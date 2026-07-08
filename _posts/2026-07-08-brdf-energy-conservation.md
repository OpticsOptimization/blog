---
layout: post
title: "BRDF Energy Conservation"
thumbnail_svg: >
  <svg viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
    <!-- Dark background implicit from currentcolor use on white elements -->
    <!-- Surface/Horizon line -->
    <line x1="0" y1="100" x2="200" y2="100" stroke="currentColor" stroke-width="2"/>
  
    <!-- Incoming light ray -->
    <line x1="50" y1="50" x2="90" y2="90" stroke="#ffffff" stroke-width="1.5" opacity="0.8"/>
    <circle cx="50" cy="50" r="2" fill="#ffffff" opacity="0.8"/>
  
    <!-- Reflection (Specular) -->
    <line x1="90" y1="90" x2="150" y2="50" stroke="#ffffff" stroke-width="1.5" opacity="0.6"/>
  
    <!-- Refraction/Absorption (Diffuse path) -->
    <line x1="90" y1="90" x2="90" y2="150" stroke="#ffffff" stroke-width="1.0" opacity="0.3"/>
    <line x1="90" y1="90" x2="70" y2="130" stroke="#ffffff" stroke-width="1.0" opacity="0.3"/>
    <line x1="90" y1="90" x2="110" y2="130" stroke="#ffffff" stroke-width="1.0" opacity="0.3"/>
  
    <!-- Subtle 'F' for Fresnel, hinting at the split -->
    <path d="M120 150 L120 120 L140 120 M120 135 L135 135" stroke="#ffffff" stroke-width="0.5" opacity="0.2" fill="none"/>
    
    <!-- Subtle 'E' for Energy, hinting at conservation -->
    <path d="M60 150 L60 120 L80 120 M60 135 L75 135 M60 150 L75 150" stroke="#ffffff" stroke-width="0.5" opacity="0.2" fill="none"/>
  
  </svg>
---

## The Unseen Architecture of Light: Mastering BRDF Energy Conservation for Realistic Shading

As a Senior Rendering Engineer, few principles are as fundamental to achieving photorealistic graphics as Physically Based Rendering (PBR). And at the heart of PBR lies the concept of energy conservation within your Bidirectional Reflectance Distribution Function (BRDF). Without it, your carefully crafted scenes devolve into fantastical realms where surfaces mysteriously generate light, breaking the illusion of reality.

The pain point I frequently encounter, especially from those transitioning into PBR, is precisely this: "How do I write a basic energy-conserving BRDF shader?" It's not just about picking the right microfacet model; it's about understanding how light interacts with a surface at a fundamental level and translating that physics into your shader code. Let's demystify this critical aspect of modern rendering.

### The Immutable Law: What is BRDF Energy Conservation?

In essence, energy conservation dictates that a surface cannot reflect more light than it receives. Light energy incident on a surface must be either reflected, absorbed, or transmitted (refracted). It cannot be created.

For a BRDF, this translates to a mathematical constraint: the integral of the BRDF over the outgoing hemisphere for any given incoming light direction must be less than or equal to 1. If it's greater than 1, you're creating light; if it's much less than 1, your surface is absorbing more than it should, potentially appearing unnaturally dark or dull.

This principle is especially crucial in microfacet BRDFs, where the surface is modeled as a collection of microscopic facets, each behaving like a tiny mirror. The overall reflection properties emerge from the statistical distribution of these facets.

### The Microfacet Paradigm: D, G, F and the Conservation Link

A typical microfacet BRDF is often expressed as:

$$ f_r(\mathbf{l}, \mathbf{v}) = \frac{D(\mathbf{h})G(\mathbf{l}, \mathbf{v}, \mathbf{h})F(\mathbf{l}, \mathbf{h})}{4 (\mathbf{n} \cdot \mathbf{l}) (\mathbf{n} \cdot \mathbf{v})} $$

Where:
*   $D(\mathbf{h})$ is the **Normal Distribution Function** (NDF), describing the distribution of microfacet normals.
*   $G(\mathbf{l}, \mathbf{v}, \mathbf{h})$ is the **Geometry Function** (or masking-shadowing function), accounting for self-occlusion of microfacets.
*   $F(\mathbf{l}, \mathbf{h})$ is the **Fresnel Term**, determining the fraction of light reflected at a given angle of incidence.
*   $(\mathbf{n} \cdot \mathbf{l})$ and $(\mathbf{n} \cdot \mathbf{v})$ are dot products of the surface normal $\mathbf{n}$ with the light vector $\mathbf{l}$ and view vector $\mathbf{v}$ respectively, handling projected area.
*   $\mathbf{h}$ is the halfway vector, $\mathbf{h} = \text{normalize}(\mathbf{l} + \mathbf{v})$.

While each component plays a vital role, the Fresnel term is the primary gatekeeper for energy conservation, particularly when linking the specular and diffuse components.

### Fresnel: The Gatekeeper of Reflectance

The Fresnel term dictates how much light is *reflected* from a surface versus how much is *refracted* (or absorbed, in the case of opaque materials). It's dependent on the angle between the incoming light ray and the surface normal (or more accurately, the microfacet normal), as well as the material's index of refraction (IOR).

A common approximation, attributed to Schlick, is widely used for its simplicity and good visual fidelity:

$$ F_{Schlick}(F_0, \cos \theta) = F_0 + (1 - F_0)(1 - \cos \theta)^5 $$

Where:
*   $F_0$ is the **base reflectance at normal incidence** (when light hits perpendicular, $\cos \theta = 1$). For dielectrics (non-metals), $F_0$ is typically a low scalar value (e.g., 0.04 for common plastics). For metals, $F_0$ is often a colored value representing the metal's inherent metallic color.
*   $\cos \theta$ is the cosine of the angle of incidence, often taken as $(\mathbf{l} \cdot \mathbf{h})$ for specular reflections or $(\mathbf{n} \cdot \mathbf{v})$ for generalized outgoing reflections.

Crucially, the Fresnel term ensures that as the angle of incidence increases (grazing angles), more light is reflected specularly, regardless of the material type.

### The Energy Conservation Link: Connecting Specular and Diffuse

Here's where the rubber meets the road for writing an energy-conserving BRDF. For *dielectric* materials, light that is *not* reflected specularly by the surface (as determined by Fresnel) is available to be scattered diffusely *or* absorbed. For *metals*, virtually all light is reflected specularly at the surface; there is no diffuse component.

The key insight for energy conservation is to use the Fresnel term to modulate the diffuse component:

$$ \text{Diffuse Reflectivity} = (1 - F) \times \text{Base Color (Albedo)} \times (1 - \text{Metallic}) $$

The $(1 - F)$ factor represents the fraction of light that *penetrates* the surface rather than being reflected specularly. This fraction is then available for diffuse scattering. For metallic materials, the `(1 - Metallic)` factor ensures the diffuse component is correctly extinguished, as metals have no subsurface scattering.

### Building a Basic Energy-Conserving BRDF Shader (HLSL/GLSL Concept)

Let's put these concepts into a conceptual shader structure. We'll assume inputs like `N` (surface normal), `V` (view vector), `L` (light vector), `baseColor` (albedo), `metallic` (0 for dielectric, 1 for metal), and `roughness`.

```hlsl
// Common PBR shader inputs
float3 N;         // Surface normal
float3 V;         // View vector
float3 L;         // Light vector
float3 baseColor; // Base albedo / F0 for metals
float metallic;   // Metallicness factor (0.0 for dielectric, 1.0 for metal)
float roughness;  // Surface roughness

// Constants
const float PI = 3.1415926535;
const float3 F0_DIELECTRIC = float3(0.04, 0.04, 0.04); // Default F0 for dielectrics

// Helper functions (placeholder, implementation details follow)
float D_GGX(float NdotH, float alpha);
float G_SmithSchlickGGX(float NdotL, float NdotV, float alpha);
float3 Fresnel_Schlick(float3 F0, float cosTheta);

float3 CalculateEnergyConservingBRDF(float3 N, float3 V, float3 L, 
                                     float3 baseColor, float metallic, float roughness) {
    // Calculate Halfway vector
    float3 H = normalize(V + L);

    // Clamp dot products to avoid negative values
    float NdotL = max(0.001, dot(N, L)); // Epsilon to prevent division by zero
    float NdotV = max(0.001, dot(N, V));
    float NdotH = max(0.0, dot(N, H));
    float LdotH = max(0.0, dot(L, H)); // Or VdotH, they are equal if V, L, H are normalized.

    // 1. Determine F0 for the material
    // For dielectrics, F0 is constant. For metals, F0 IS the baseColor.
    float3 F0_material = lerp(F0_DIELECTRIC, baseColor, metallic);

    // 2. Calculate Fresnel term (for specular reflection)
    // We use LdotH here as the cosine of the angle between light and microfacet normal.
    float3 F = Fresnel_Schlick(F0_material, LdotH);

    // 3. Diffuse Component (Lambertian with Energy Conservation)
    // The amount of light available for diffuse scattering is (1 - F).
    // For metals (metallic=1), this factor becomes (1-F)*0, correctly eliminating diffuse.
    float3 kD = (1.0 - F) * (1.0 - metallic);
    float3 diffuseBRDF = kD * baseColor / PI; // BaseColor here represents the diffuse albedo

    // 4. Specular Component (Microfacet BRDF)
    float alpha = roughness * roughness; // Convert roughness to alpha for GGX

    // Distribution (D) - GGX Trowbridge-Reitz
    float D = D_GGX(NdotH, alpha);

    // Geometry (G) - Smith's method with Schlick-GGX approximation
    float G = G_SmithSchlickGGX(NdotL, NdotV, alpha);

    // Final specular BRDF term
    float3 specularBRDF = (D * G * F) / (4.0 * NdotL * NdotV);

    // Summing diffuse and specular for the final BRDF
    // Note: We multiply by NdotL later when applying direct lights,
    // so this is just the BRDF value itself.
    return diffuseBRDF + specularBRDF;
}

// *** Helper Function Implementations ***

// Schlick's Fresnel Approximation
float3 Fresnel_Schlick(float3 F0, float cosTheta) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}

// GGX Trowbridge-Reitz Normal Distribution Function
float D_GGX(float NdotH, float alpha) {
    float alpha2 = alpha * alpha;
    float NdotH2 = NdotH * NdotH;
    float denom = (NdotH2 * (alpha2 - 1.0) + 1.0);
    return alpha2 / (PI * denom * denom);
}

// Smith's Geometry Function with Schlick-GGX approximation
float G_SchlickGGX_partial(float NdotX, float alpha) {
    float k = (alpha * 0.5); // For Direct Light
    return NdotX / (NdotX * (1.0 - k) + k);
}

float G_SmithSchlickGGX(float NdotL, float NdotV, float alpha) {
    return G_SchlickGGX_partial(NdotL, alpha) * G_SchlickGGX_partial(NdotV, alpha);
}
```

This shader structure directly implements the energy conservation principle by using the `(1.0 - F)` term to modulate the diffuse component based on the amount of light reflected specularly. For metals, the `(1.0 - metallic)` factor correctly turns off the diffuse term entirely, as light that isn't specularly reflected is absorbed by the metal, not diffusely scattered.

### Beyond the Basics: Refining Conservation and Further Reading

While the above provides a solid, energy-conserving foundation for direct lighting, the journey into accurate PBR is rich with further considerations:

*   **Average Fresnel for Diffuse:** For truly rigorous diffuse energy conservation (especially for image-based lighting), the diffuse component should ideally be modulated by the *average* Fresnel term over the hemisphere, not just the specific $F$ for the current light direction. Approximations exist (e.g., from Epic Games) to calculate this more accurately.
*   **Split Sum Approximation:** For pre-filtered environment maps (IBL), the diffuse and specular contributions are typically pre-computed separately using a split-sum approximation, each needing to maintain energy conservation within its own domain.
*   **Clear Coats and Multi-Layered Materials:** These introduce additional layers of Fresnel and energy distribution complexities, requiring careful management to avoid creating light.

For those ready to dive deeper into the mathematical rigor, advanced approximations, and practical implementations of these concepts, including detailed derivations of D and G functions, and comprehensive strategies for integrating BRDFs into full rendering pipelines, *The Art and Science of Physically Based Rendering* offers an unparalleled resource. It meticulously deconstructs the physics and the computational techniques, moving beyond introductory explanations to equip you with the tools for production-grade rendering.

### Conclusion

Energy conservation is not merely an aesthetic choice in PBR; it's a physical mandate that underpins the realism of your rendered scenes. By understanding the roles of Fresnel and the interplay between specular and diffuse components, you can craft shaders that faithfully represent light interaction. The basic shader provided here demonstrates this fundamental principle, serving as a robust starting point for your physically based rendering endeavors. Keep experimenting, keep learning, and your scenes will shine with newfound authenticity.

---

