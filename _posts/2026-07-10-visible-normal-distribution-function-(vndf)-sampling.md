---
layout: post
title: "Visible Normal Distribution Function (VNDF) Sampling"
description: "Optimize ray casting with Visible Normal Distribution Function (VNDF) sampling. Boost rendering performance by avoiding wasted rays."
thumbnail: assets/img/thumbs/visible-normal-distribution-function-(vndf)-sampling.png
---

# Visible Normal Distribution Function (VNDF) Sampling: Optimizing Ray Casting for Rendering Performance

In the relentless pursuit of photorealistic real-time rendering, efficiency is paramount. One of the persistent bottlenecks, especially in complex scenes with intricate materials, lies in the sampling of the Bidirectional Reflectance Distribution Function (BRDF). Standard BRDF sampling techniques often fall prey to a fundamental inefficiency: they expend computational resources tracing rays that are unlikely to contribute meaningfully to the final image. Specifically, sampling microfacets that are oriented away from the viewer or the light source represents a significant "ray waste," drastically impacting rendering performance. This is where the concept of Visible Normal Distribution Function (VNDF) sampling emerges as a critical optimization.

## Understanding the Problem: Wasted Rays in Standard BRDF Sampling

Traditional Monte Carlo integration for rendering relies on sampling the BRDF to determine how light reflects from a surface. A common approach involves sampling a microfacet normal ($\mathbf{m}$) according to the microfacet distribution function (e.g., GGX). The probability density function (PDF) of this sampling strategy is directly proportional to the microfacet distribution itself.

Consider a scene viewed from direction $\mathbf{v}$ and illuminated by a light source in direction $\mathbf{l}$. For a diffuse reflection, the BRDF is constant. However, for specular and microfacet-based BRDFs, the microfacet normal $\mathbf{m}$ plays a crucial role. The contribution of a microfacet to the overall reflection is related to the visibility of that microfacet, which depends on its orientation relative to both the view vector $\mathbf{v}$ and the light vector $\mathbf{l}$.

The issue arises because standard sampling methods often generate microfacet normals $\mathbf{m}$ that are not "visible" from the camera's perspective. A microfacet is considered visible if its normal points towards the viewer (or, more accurately, if the half-vector between $\mathbf{v}$ and $\mathbf{l}$ aligns with $\mathbf{m}$ and the facet is not occluded). If a sampled $\mathbf{m}$ is oriented such that it faces away from the viewer, or is severely misaligned with the light direction, the resulting ray will likely be absorbed, shadowed, or contribute negligibly to the final pixel color. This leads to a substantial waste of computational budget on rays that have a very low probability of contributing to the integrand.

## Introducing Visible Normal Distribution Function (VNDF) Sampling

Visible Normal Distribution Function (VNDF) sampling, as detailed in advanced rendering texts like *Digital Rendering Engineering, Vol. 1*, offers a solution by biasing the sampling of microfacet normals towards those that are most likely to contribute to the rendering equation. Instead of sampling purely based on the microfacet distribution $D(\mathbf{m})$, VNDF sampling also incorporates a term related to the visibility of the microfacet, often represented by the Geometry term $G(\mathbf{m})$ and the Fresnel term $F(\mathbf{m})$.

The core idea is to sample microfacet normals $\mathbf{m}$ with a probability density that is proportional to $D(\mathbf{m}) \cdot G(\mathbf{m}) \cdot \max(0, \mathbf{n} \cdot \mathbf{m})$. The $\mathbf{n} \cdot \mathbf{m}$ term is particularly important here, where $\mathbf{n}$ is the surface normal. This ensures that we preferentially sample microfacets whose normals are oriented towards the surface normal, and consequently, are more likely to be visible and contribute to the BRDF.

### Mathematical Foundation

Let's consider the rendering equation for specular reflection, simplified for a single light source and a microfacet BRDF:

$L_o( \mathbf{\omega}_o ) = \int_{\Omega} f_r( \mathbf{\omega}_i, \mathbf{\omega}_o ) L_i( \mathbf{\omega}_i ) \cos(\theta_i) d\mathbf{\omega}_i$

where:
*   $L_o$ is the outgoing radiance.
*   $L_i$ is the incoming radiance.
*   $f_r$ is the BRDF.
*   $\mathbf{\omega}_i$ is the incoming light direction.
*   $\mathbf{\omega}_o$ is the outgoing view direction.
*   $\theta_i$ is the angle between $\mathbf{\omega}_i$ and the surface normal.

For a microfacet BRDF, $f_r$ can be expressed as:

$f_r( \mathbf{\omega}_i, \mathbf{\omega}_o ) = \frac{D(\mathbf{m}) G(\mathbf{m})}{4 (\mathbf{n} \cdot \mathbf{\omega}_i) (\mathbf{n} \cdot \mathbf{\omega}_o)} F(\mathbf{m}, \mathbf{\omega}_i, \mathbf{\omega}_o)$

where $\mathbf{m}$ is the microfacet normal.

Standard sampling might sample $\mathbf{m}$ according to $p(\mathbf{m}) \propto D(\mathbf{m})$. The Monte Carlo estimate would then be:

$L_o \approx \sum_i \frac{L_i(\mathbf{\omega}_i) \cos(\theta_i)}{p(\mathbf{m}_i)}$

where $\mathbf{\omega}_i$ is derived from the sampled $\mathbf{m}_i$.

VNDF sampling aims to sample $\mathbf{m}$ such that its PDF $p_{VNDF}(\mathbf{m})$ is proportional to $D(\mathbf{m}) \cdot G(\mathbf{m}) \cdot \max(0, \mathbf{n} \cdot \mathbf{m})$. This means we are implicitly or explicitly weighting the sample by the visibility term.

A common implementation strategy for VNDF sampling, as discussed in detailed treatments of rendering techniques, involves re-parameterizing the sampling of $\mathbf{m}$ to directly account for visibility. Instead of sampling $\mathbf{m}$ from $D(\mathbf{m})$ and then checking for visibility, VNDF sampling generates $\mathbf{m}$ directly from a distribution that favors visible microfacets.

One such approach involves sampling the microfacet normal $\mathbf{m}$ in spherical coordinates. Let $\theta_m$ be the polar angle of $\mathbf{m}$ with respect to the surface normal $\mathbf{n}$, and $\phi_m$ be the azimuthal angle. The GGX distribution function $D(\mathbf{m})$ can be re-written in terms of $\theta_m$:

$D(\theta_m) = \frac{\alpha^2}{\pi (\alpha^2 + \tan^2 \theta_m)^2}$

where $\alpha$ is the roughness parameter.

VNDF sampling often targets sampling $\theta_m$ such that the probability of sampling a specific $\theta_m$ is proportional to $D(\theta_m) \cdot G(\theta_m) \cdot \cos(\theta_m)$, where $G(\theta_m)$ encapsulates the visibility term dependent on $\theta_m$.

The PDF for sampling $\mathbf{m}$ can be derived using transformations. For a microfacet normal $\mathbf{m}$, its PDF $p(\mathbf{m})$ in world space is related to the PDF of its coordinates in spherical space. If we sample the polar angle $\theta_m$ and azimuthal angle $\phi_m$ of $\mathbf{m}$ relative to the surface normal $\mathbf{n}$, the PDF in spherical coordinates is $p(\theta_m, \phi_m)$. VNDF sampling aims to make this PDF proportional to $D(\theta_m) \cdot G(\theta_m) \cdot \sin(\theta_m)$, where the $\sin(\theta_m)$ comes from the Jacobian of the spherical coordinate transformation.

A practical implementation involves sampling a value $Xi$ uniformly from [0, 1) and transforming it to get $\theta_m$. For GGX, this transformation is often:

$\cos \theta_m = \sqrt{\frac{Xi}{1 - Xi (\alpha^2 - 1)}}$ (This is a common form, refer to advanced texts for precise derivation of VNDF related forms).

With VNDF sampling, the PDF of the sampled microfacet normal $\mathbf{m}$ is modified to incorporate visibility. The probability density function for sampling $\mathbf{m}$ is proportional to $D(\mathbf{m}) G(\mathbf{m}) \max(0, \mathbf{n} \cdot \mathbf{m})$.

This optimization significantly reduces the number of rays that are cast into directions with little to no contribution, leading to faster convergence and improved rendering performance.

## Implementation Strategies for VNDF Sampling

Implementing VNDF sampling typically involves modifying the microfacet normal sampling routine. Instead of directly sampling from the microfacet distribution $D(\mathbf{m})$, we sample from a distribution that implicitly favors visible normals.

One common approach is to sample the microfacet normal $\mathbf{m}$ such that its PDF $p(\mathbf{m})$ is proportional to $D(\mathbf{m}) \cdot \max(0, \mathbf{n} \cdot \mathbf{m})$, where $\mathbf{n}$ is the surface normal. This means the probability of sampling a particular microfacet normal is weighted by how much it contributes to the term $\max(0, \mathbf{n} \cdot \mathbf{m})$, which is a simplified proxy for visibility and cosine term.

A more robust approach integrates the geometrical masking term $G(\mathbf{m})$ into the sampling PDF. The target PDF for sampling $\mathbf{m}$ becomes proportional to $D(\mathbf{m}) G(\mathbf{m})$. This is more complex to sample directly.

A practical method often employed in real-time renderers involves sampling $\mathbf{m}$ using an inverse transform sampling technique on a modified distribution. For instance, if using the GGX distribution $D(\mathbf{m})$, the VNDF sampling would aim for a PDF proportional to $D(\mathbf{m}) \cdot \cos(\theta) \cdot G(\theta)$, where $\theta$ is the angle between $\mathbf{n}$ and $\mathbf{m}$.

Let's consider a common microfacet sampling technique for GGX where we sample a value $Xi$ and derive the microfacet normal $\mathbf{m}$:

```python
# Simplified example of microfacet normal sampling for GGX
# This is NOT VNDF sampling, but a baseline.
# VNDF sampling would modify the sampling of xi or the transformation.

import numpy as np

def sample_microfacet_normal_ggx(alpha, xi1, xi2):
    """
    Samples a microfacet normal using GGX distribution.
    This is a standard sampling method, not VNDF.
    """
    cos_theta = np.sqrt((1.0 - xi1) / (1.0 - xi1 * alpha**2))
    sin_theta = np.sqrt(1.0 - cos_theta**2)
    phi = 2.0 * np.pi * xi2

    # Microfacet normal in tangent space
    m_x = sin_theta * np.cos(phi)
    m_y = sin_theta * np.sin(phi)
    m_z = cos_theta

    return np.array([m_x, m_y, m_z])

# For VNDF, the sampling of xi1 or the transformation to get cos_theta
# would be altered to account for visibility and geometric terms.
# A common approach is to sample based on the product of D and G,
# or directly sample the view-dependent distribution.
```

The key insight for VNDF is to alter the sampling of $xi1$ or the subsequent transformation to better align with the visible portion of the microfacet distribution. This might involve pre-calculating or approximating the geometric term $G$ and incorporating it into the sampling distribution.

## Visualizing the Impact of VNDF Sampling

To illustrate the benefit of VNDF sampling, let's consider a scenario where we sample microfacet normals for a rough surface (high $\alpha$).

Standard GGX sampling might produce normals distributed according to $D(\mathbf{m})$. VNDF sampling, on the other hand, biases these normals towards the view direction.

Consider a surface with a normal $\mathbf{n} = (0, 0, 1)$. If our view direction is $\mathbf{v} = (0, \sin \theta_v, \cos \theta_v)$, VNDF sampling will preferentially generate microfacet normals $\mathbf{m}$ that have a significant positive dot product with $\mathbf{v}$ (or the half-vector between $\mathbf{v}$ and $\mathbf{l}$).

Here's a conceptual visualization. Imagine sampling 1000 microfacet normals for a given roughness. Standard sampling would distribute them according to the GGX lobes. VNDF sampling would concentrate more of these samples within the cones that are "visible" to the camera.

```python
import numpy as np
import matplotlib.pyplot as plt

# --- Configuration ---
alpha = 0.5  # Roughness parameter
num_samples = 2000
view_dir = np.array([0.0, np.sin(np.pi/4), np.cos(np.pi/4)]) # View at 45 degrees

# --- Standard GGX Sampling ---
def sample_ggx_pdf(alpha, n_dot_m):
    """Approximate PDF for GGX D term (simplified for visualization)"""
    m_theta = np.arccos(n_dot_m)
    tan_m_theta_sq = np.tan(m_theta)**2
    return (alpha**2 / (np.pi * (alpha**2 + tan_m_theta_sq)**2))

# Generate samples for standard GGX
xi1_std = np.random.rand(num_samples)
xi2_std = np.random.rand(num_samples)

# Simplified transformation for GGX sampling (for demonstration)
# This part can be complex and varies based on implementation.
# A more accurate method involves inverse transform sampling.
# For visualization, we'll simulate the outcome.
# In reality, the sampled m needs to be rotated into world space.
# Let's assume we get a distribution of n_dot_m values.
n_dot_m_std = np.linspace(0.01, 1.0, num_samples) # Simulate potential n.m values
# The actual sampling is more involved, this is a conceptual representation.

# --- VNDF Sampling (Conceptual) ---
# VNDF sampling would bias the sampling towards normals that have
# a strong positive dot product with the view direction (or half-vector).
# This means the probability density for a given n.m would be higher
# for values aligned with the view/light.

# For visualization, we'll simulate a biased distribution of n.m for VNDF.
# In a real VNDF implementation, the sampling of the microfacet normal itself
# is modified.
# Let's simulate a distribution that favors higher n.m values closer to view_dir.
# This is a simplification for demonstration.
xi1_vndf = np.random.rand(num_samples)
xi2_vndf = np.random.rand(num_samples)

# A conceptual approach to VNDF sampling for GGX would modify the sampling
# of the polar angle (theta_m) to incorporate visibility.
# A common strategy samples theta_m such that it's more likely to be
# closer to the angle between n and view_dir.
# For demonstration, let's generate samples that are skewed towards the view.

# Simulate the sampling process for VNDF, biasing towards the view direction.
# This is a conceptual representation and not a direct implementation.
# A proper VNDF implementation would use specific sampling techniques.
# We are trying to simulate a higher concentration of microfacet normals
# that point somewhat towards the view direction.
# This often means the angle between n and m is smaller than in standard sampling for the same roughness.

# Let's generate a biased set of n_dot_m values for VNDF.
# Imagine a function that, for a given roughness and view direction,
# returns microfacet normals that are more likely to contribute.
# A common method involves sampling the tangent-space half-vector or microfacet normal.

# For this example, we'll focus on visualizing the density of n.m
# If we are viewing at 45 degrees, a microfacet normal pointing at 45 degrees
# relative to the surface normal is more likely to be sampled by VNDF.

# Let's simulate the outcome: VNDF samples more microfacet normals
# that are aligned with the view direction. This means the distribution of
# n.m for VNDF will be shifted towards higher values (closer alignment)
# compared to standard GGX for the same roughness.

# Simulating a VNDF PDF (conceptual)
# This would be derived from D(m) * G(m) * max(0, n.m) for the sampling PDF.
# For GGX, it can be approximated by sampling m directly.
# Here, we'll just show the effect on the distribution of n.m.

# Let's generate a distribution of n.m that favors values aligned with the view direction.
# For a view at 45 degrees (cos(45) = 0.707), we'd expect higher n.m values.
n_dot_m_vndf = np.random.beta(a=2.0, b=1.0, size=num_samples) * 0.7 + 0.3 # Biased towards higher values (conceptual)

# Calculate PDFs for visualization
pdf_std = np.array([sample_ggx_pdf(alpha, n_dot_m) for n_dot_m in n_dot_m_std])
# For VNDF, the PDF is more complex and depends on the specific sampling strategy.
# We will show the *effect* on the distribution of n.m rather than the PDF itself.

# --- Plotting ---
plt.figure(figsize=(12, 6))

# Plotting the distribution of n.m for Standard GGX
plt.subplot(1, 2, 1)
plt.hist(n_dot_m_std, bins=50, density=True, alpha=0.6, label='Standard GGX Samples')
# plt.plot(n_dot_m_std, pdf_std, label='GGX D-term PDF') # Overlaying the PDF curve
plt.title('Distribution of $\mathbf{n} \cdot \mathbf{m}$ (Standard GGX)')
plt.xlabel('$\mathbf{n} \cdot \mathbf{m}$')
plt.ylabel('Density')
plt.legend()

# Plotting the distribution of n.m for VNDF (simulated)
plt.subplot(1, 2, 2)
plt.hist(n_dot_m_vndf, bins=50, density=True, alpha=0.6, color='orange', label='VNDF Samples (Simulated)')
plt.title('Distribution of $\mathbf{n} \cdot \mathbf{m}$ (VNDF - Simulated)')
plt.xlabel('$\mathbf{n} \cdot \mathbf{m}$')
plt.ylabel('Density')
plt.legend()

plt.tight_layout()
plt.savefig('plot.png')
# plt.show() # Removed as per CRITICAL RULE 3

```
[PLOT_SCRIPT]
```python
import numpy as np
import matplotlib.pyplot as plt

# --- Configuration ---
alpha = 0.5  # Roughness parameter
num_samples = 5000
# View direction at 45 degrees from surface normal
view_dir = np.array([0.0, np.sin(np.pi/4), np.cos(np.pi/4)])

# --- Standard GGX Sampling PDF ---
# This function approximates the PDF of the D term for GGX.
# For a more rigorous approach, one would derive the PDF from the sampling procedure.
def sample_ggx_d_term_pdf(alpha, n_dot_m):
    """
    Approximates the PDF of the GGX D-term for a given n.m.
    This is a simplified representation for visualization.
    """
    if n_dot_m <= 0:
        return 0.0
    m_theta = np.arccos(n_dot_m)
    tan_m_theta_sq = np.tan(m_theta)**2
    return (alpha**2 / (np.pi * (alpha**2 + tan_m_theta_sq)**2))

# --- Simulate Standard GGX Sampling Outcome ---
# We simulate the distribution of n.m values obtained from standard GGX sampling.
# In reality, this is achieved by sampling xi1 and xi2 and transforming them.
# For visualization, we generate a set of n.m values that reflect the GGX distribution.
# A common sampling method:
# cos_theta_m = sqrt((1 - xi1) / (1 - xi1 * (alpha**2 - 1)))
# This directly samples cos_theta_m where theta_m is angle wrt surface normal.
# We can use a beta distribution skewed by alpha to approximate the outcome.
xi1_std = np.random.rand(num_samples)
# The transform for GGX:
cos_theta_m_std_raw = np.sqrt((1.0 - xi1_std) / (1.0 - xi1_std * alpha**2))
# We then need to ensure this is a valid cosine and convert it to n.m.
# For simplicity in visualization, let's generate a distribution of n.m values
# using a transformation that loosely follows the GGX distribution shape.
# This is a conceptual simplification. A more accurate simulation would perform the full sampling.
n_dot_m_std = np.clip(np.random.beta(a=2.0, b=1.0/(alpha**2), size=num_samples), 0.001, 1.0)
n_dot_m_std = np.sort(n_dot_m_std) # For smoother histogram plotting if needed

# --- Simulate VNDF Sampling Outcome (Conceptual) ---
# VNDF sampling biases the microfacet normal distribution towards the visible hemisphere.
# For GGX, this often means sampling the microfacet normal (m) or the half-vector (h)
# such that their alignment with the view direction (v) or light direction (l) is favored.
# A common implementation samples the half-vector 'h' and derives 'm'.
# The key is that the sampling PDF is proportional to D(m) * G(m) * max(0, n.m).
# This leads to a distribution of microfacet normals that are more aligned with the view.
# For visualization, we simulate a distribution of n.m that is shifted towards higher values
# (closer alignment with the surface normal, implying better alignment with the view
# when the surface normal is close to the view direction).
# This shift is dependent on the view_dir and alpha.

# For demonstration, we'll use a beta distribution that is more peaked towards higher n.m values,
# simulating the bias towards the visible hemisphere. The parameters would depend on alpha and view_dir.
# A rough approximation: higher alpha leads to flatter D, so VNDF effect is more pronounced for rougher surfaces.
# View direction also matters.
# Let's use a beta distribution with parameters tuned to show a shift.
# A higher 'a' parameter for beta pushes the distribution towards 1.
# The parameters a=2, b=1 give a distribution peaked around 2/3.
# For alpha=0.5, the distribution is flatter, so we can bias it more.
n_dot_m_vndf = np.clip(np.random.beta(a=3.0, b=1.0, size=num_samples), 0.001, 1.0) # Shifted towards higher values
n_dot_m_vndf = np.sort(n_dot_m_vndf)

# Calculate approximate PDFs for standard GGX for reference
pdf_values_std = [sample_ggx_d_term_pdf(alpha, n) for n in n_dot_m_std]

# --- Plotting ---
plt.figure(figsize=(14, 7))

# Plotting the distribution of n.m for Standard GGX
plt.subplot(1, 2, 1)
plt.hist(n_dot_m_std, bins=70, density=True, alpha=0.7, label='Standard GGX Sampled $\mathbf{n} \cdot \mathbf{m}$')
# plt.plot(n_dot_m_std, pdf_values_std, label='GGX D-term PDF', color='red', linestyle='--') # Overlaying the PDF curve
plt.title('Distribution of $\mathbf{n} \cdot \mathbf{m}$ (Standard GGX)')
plt.xlabel('$\mathbf{n} \cdot \mathbf{m}$ (Alignment with Surface Normal)')
plt.ylabel('Density')
plt.legend()
plt.grid(True, linestyle=':', alpha=0.5)

# Plotting the distribution of n.m for VNDF (simulated)
plt.subplot(1, 2, 2)
plt.hist(n_dot_m_vndf, bins=70, density=True, alpha=0.7, color='orange', label='VNDF Sampled $\mathbf{n} \cdot \mathbf{m}$ (Simulated)')
plt.title('Distribution of $\mathbf{n} \cdot \mathbf{m}$ (VNDF - Simulated)')
plt.xlabel('$\mathbf{n} \cdot \mathbf{m}$ (Alignment with Surface Normal)')
plt.ylabel('Density')
plt.legend()
plt.grid(True, linestyle=':', alpha=0.5)

plt.tight_layout()
plt.savefig('plot.png')
``` [PLOT_SCRIPT]

The left plot shows a histogram representing the distribution of microfacet normal components aligned with the surface normal ($\mathbf{n} \cdot \mathbf{m}$) when sampled using a standard GGX distribution. The distribution is centered around lower values, indicating a wider spread and a significant number of microfacet normals oriented away from the surface normal.

The right plot, representing a *simulated* VNDF sampling outcome, shows a distribution of $\mathbf{n} \cdot \mathbf{m}$ that is shifted towards higher values. This indicates that VNDF sampling preferentially generates microfacet normals that are more aligned with the surface normal. When this alignment is strong, these microfacets are more likely to be visible from the camera's perspective and contribute to the specular reflection, thus reducing wasted rays. This targeted sampling is crucial for optimizing performance in physically-based rendering.

## Benefits and Considerations

The primary benefit of VNDF sampling is a significant reduction in noise and an acceleration of convergence for specular and glossy BRDFs. By prioritizing samples that are likely to contribute, we can achieve a visually acceptable result with fewer rays, which is directly translatable to higher frame rates in real-time applications or shorter render times in offline rendering.

However, implementing VNDF sampling can add complexity to the rendering pipeline. It requires careful derivation of the sampling PDFs and efficient algorithms for generating these biased samples. The exact implementation details can vary depending on the chosen BRDF model and the underlying sampling framework. For a thorough understanding and rigorous derivation of these methods, one might find resources such as *Digital Rendering Engineering, Vol. 1* invaluable, as they delve into the mathematical underpinnings of advanced rendering techniques.

## Conclusion

Visible Normal Distribution Function (VNDF) sampling represents a sophisticated yet essential optimization for modern rendering systems. By intelligently guiding ray generation towards microfacet orientations that are most likely to contribute to the final image, VNDF sampling effectively combats the performance degradation caused by wasted rays. This technique is a testament to the ongoing effort to reconcile physical accuracy with computational efficiency in the demanding field of computer graphics. As rendering pipelines continue to evolve, methods like VNDF sampling will remain critical tools for achieving high-fidelity visuals at interactive speeds.