<div align="center">
  <h1>Knotoids &amp; Conformations in Protein Folding Dynamics</h1>
  <p><b>A Computational Topology Framework for Open Polypeptide Chains</b></p>

  <img src="https://img.shields.io/badge/build-passing-brightgreen?style=flat-square" alt="Build Status">
  <img src="https://img.shields.io/badge/version-1.2.0-blue?style=flat-square" alt="Version">
  <img src="https://img.shields.io/badge/license-MIT-purple?style=flat-square" alt="License">
  <img src="https://img.shields.io/badge/python-3.9%2B-blue?style=flat-square" alt="Python">
</div>

<br>

<details>
  <summary><b>Table of Contents</b></summary>
  <ul>
    <li><a href="#overview">Overview</a></li>
    <li><a href="#mathematical-framework">Mathematical Framework</a>
      <ul>
        <li><a href="#knotoids-and-open-curve-topology">Knotoids &amp; Open Curve Topology</a></li>
        <li><a href="#topological-invariants">Topological Invariants</a></li>
      </ul>
    </li>
    <li><a href="#protein-folding-dynamics">Protein Folding Dynamics</a>
      <ul>
        <li><a href="#langevin-dynamics">Langevin Dynamics</a></li>
        <li><a href="#topological-energy-landscape">Topological Energy Landscape</a></li>
      </ul>
    </li>
    <li><a href="#computational-pipeline">Computational Pipeline</a></li>
    <li><a href="#installation-usage">Installation &amp; Usage</a></li>
    <li><a href="#references">References</a></li>
  </ul>
</details>

<hr>

<h2 id="overview">Overview</h2>

<p>Unlike closed mathematical knots, protein backbones are strictly open chains (linear polypeptides) with clear N- and C-termini. Traditional knot theory requires artificial closure schemes to analyze protein entanglement, which introduces topological artifacts.</p>

<p>This repository implements the theory of <b>Knotoids</b> (introduced by V. Turaev, 2012) to natively analyze the topology of open curves without closure. By coupling topological invariants with Molecular Dynamics (MD), this framework maps the dynamic conformational space of proteins, identifying transient topological barriers and entanglement-driven folding pathways.</p>

<hr>

<h2 id="mathematical-framework">Mathematical Framework</h2>

<h3 id="knotoids-and-open-curve-topology">Knotoids and Open Curve Topology</h3>

<p>A protein backbone conformation at time $t$ is modeled as a continuous, non-self-intersecting open curve $K(t) \subset \mathbb{R}^3$.</p>

<p>To define its topology without arbitrary closure, we project $K$ onto a 2-dimensional sphere $S^2$ (or plane $\mathbb{R}^2$). Let $v \in S^2$ be a projection vector. The projection map is defined as:</p>

$$p_v:\mathbb{R}^3\to\Sigma$$

<p>A <b>knotoid diagram</b> $K_v$ is the generic immersion of the curve into $\Sigma$, equipped with over- and under-crossing data at transverse double points.</p>

<p>Two knotoid diagrams are considered equivalent (representing the same knotoid type) if they can be related by a finite sequence of:</p>

<ol>
  <li><b>Planar isotopies</b> (deformations of the diagram that do not change the crossing structure).</li>
  <li><b>Reidemeister Moves</b> ($\Omega_1, \Omega_2, \Omega_3$), provided these moves occur strictly in the interior of the diagram and do not sweep strands across the endpoints $p_v(\text{N-terminus})$ or $p_v(\text{C-terminus})$.</li>
</ol>

<p>Because the observed topology depends heavily on the chosen projection direction $v$, a single 3D conformation does not correspond to a single knotoid, but rather a <b>knotoid spectrum</b>. The topological state is characterized by the probability distribution $P(\mathcal{K})$ over the sphere of projection directions:</p>

$$P(\mathcal{K})=\frac{1}{4\pi}\int_{S^2}\delta(K_v,\mathcal{K})dv$$

<p>where $\mathcal{K}$ represents a specific knotoid equivalence class.</p>

<h3 id="topological-invariants">Topological Invariants</h3>

<p>To classify the knotoids computationally, we utilize the extended <b>Kauffman Bracket Polynomial</b> $\langle K_v \rangle$. The polynomial is constructed via state sums over all crossings, defined recursively by the skein relations:</p>

$$\langle L_+\rangle=A\langle L_0\rangle+A^{-1}\langle L_\infty\rangle$$
$$\langle L\sqcup\bigcirc\rangle=(-A^2-A^{-2})\langle L\rangle$$

<p>Normalizing the bracket by the writhe $w(K_v)$ of the oriented diagram yields the invariant <b>Jones Polynomial</b> for the knotoid $V_{K_v}(t)$:</p>

$$V_{K_v}(t)=\left(-A^3\right)^{-w(K_v)}\langle K_v\rangle\Big|_{A=t^{-1/4}}$$

<p>This topological invariant allows us to formally distinguish between entangled motifs (e.g., slipknots, deep knotoids) and trivial open chains dynamically during the folding process.</p>

<hr>

<h2 id="protein-folding-dynamics">Protein Folding Dynamics</h2>

<h3 id="langevin-dynamics">Langevin Dynamics</h3>

<p>The folding trajectories are generated using stochastic Langevin dynamics, simulating the solvent bath at constant temperature $T$. The equations of motion for the $N$ $C_\alpha$ atoms (where $\mathbf{r}_i$ is the position of the $i$-th atom) are:</p>

$$m_i\ddot{\mathbf{r}}_i=-\nabla_{\mathbf{r}_i}U(\mathbf{r})-\zeta_i\dot{\mathbf{r}}_i+\boldsymbol{\eta}_i(t)$$

<p>Here:</p>
<ul>
  <li>$U(\mathbf{r})$ is the empirical potential energy surface (e.g., Gō-like model or AMBER force field).</li>
  <li>$\zeta_i$ is the phenomenological friction coefficient.</li>
  <li>$\boldsymbol{\eta}_i(t)$ is the stochastic thermal noise satisfying the fluctuation-dissipation theorem:</li>
</ul>

$$\langle\boldsymbol{\eta}_i(t)\cdot\boldsymbol{\eta}_j(t')\rangle=6k_BT\zeta_i\delta_{ij}\delta(t-t')$$

<h3 id="topological-energy-landscape">Topological Energy Landscape</h3>

<p>As the protein navigates $U(\mathbf{r})$, the knotoid spectrum acts as a macroscopic order parameter. An entanglement presents a topological barrier $\Delta G_{topo}^{\ddagger}$. The effective free energy landscape $F(Q, \mathcal{K})$ as a function of the fraction of native contacts $Q$ and dominant knotoid class $\mathcal{K}$ is defined by:</p>

$$F(Q,\mathcal{K})=-k_BT\ln\int\delta(Q(\mathbf{r})-Q)\delta(\text{dom}(K_v)-\mathcal{K})e^{-U(\mathbf{r})/k_BT}d\mathbf{r}$$

<p>This framework allows us to identify kinetically trapped intermediate states where the protein has established a high degree of native contacts ($Q \to 1$) but is trapped in a non-native knotoid state $\mathcal{K}_{trap} \neq \mathcal{K}_{native}$.</p>

<hr>

<h2 id="computational-pipeline">Computational Pipeline</h2>

<div align="center">
  <table>
    <thead>
      <tr>
        <th align="center">Module</th>
        <th align="center">Function</th>
        <th align="center">Algorithm Complexity</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td align="center"><code>ProjectionEngine</code></td>
        <td align="center">Generates $N_{v}$ projection diagrams over the Gaussian sphere.</td>
        <td align="center">$O(N_{v} \cdot n^2)$</td>
      </tr>
      <tr>
        <td align="center"><code>KnotoidClassifier</code></td>
        <td align="center">Calculates modified Jones polynomials via skein trees.</td>
        <td align="center">$O(2^c)$ where $c = \text{crossings}$</td>
      </tr>
      <tr>
        <td align="center"><code>DynamicsIntegrator</code></td>
        <td align="center">Runs parallel Langevin dynamics (C++ core, Python wrapped).</td>
        <td align="center">$O(N \log N)$ per step</td>
      </tr>
    </tbody>
  </table>
</div>

<hr>

<h2 id="implementation">Standalone Implementation</h2>

<p>To demonstrate the core topological calculations without requiring complex external dependencies, the following standalone Python script calculates the <b>discrete 3D Writhe</b> of a polypeptide backbone.</p> 

<p>The writhe $w(K)$ is approximated via the double sum over all non-adjacent segment pairs (a discrete analogue of the Gauss linking integral). This structural parameter is a fundamental component for normalizing the Kauffman bracket in knotoid evaluation.</p>

<h3 id="python-code">Python Topological Analyzer</h3>

<pre><code class="language-python">import numpy as np

def generate_trefoil_backbone(n_points=150):
    """Generates a synthetic 3D open curve (trefoil knotoid motif)."""
    t = np.linspace(0.1, 2 * np.pi - 0.1, n_points)
    x = np.sin(t) + 2 * np.sin(2 * t)
    y = np.cos(t) - 2 * np.cos(2 * t)
    z = -np.sin(3 * t)
    return np.column_stack((x, y, z))

def compute_discrete_writhe(coords):
    """
    Computes the writhe of a 3D curve using the discrete Gauss integral.
    Formula: Wr = (1 / 4*pi) * sum( (dr_i x dr_j) \cdot r_ij / |r_ij|^3 )
    """
    n = len(coords)
    writhe = 0.0
    
    # Calculate tangent vectors (dr) for each segment
    dr = np.diff(coords, axis=0)
    
    # Midpoints of each segment for distance calculations
    midpoints = (coords[:-1] + coords[1:]) / 2.0
    
    for i in range(n - 1):
        for j in range(i + 2, n - 1): # Skip adjacent segments to avoid singularities
            # Vector between segment midpoints
            r_ij = midpoints[i] - midpoints[j]
            dist = np.linalg.norm(r_ij)
            
            if dist > 1e-6:
                # Cross product of tangent vectors
                cross_prod = np.cross(dr[i], dr[j])
                
                # Scalar triple product divided by distance cubed
                solid_angle = np.dot(cross_prod, r_ij) / (dist ** 3)
                writhe += solid_angle
                
    return writhe / (4 * np.pi)

if __name__ == "__main__":
    print("Initializing topological analysis...")
    
    # 1. Generate conformation (e.g., from MD trajectory)
    backbone = generate_trefoil_backbone(200)
    
    # 2. Compute topological invariant
    wr = compute_discrete_writhe(backbone)
    
    print(f"Total Backbone Atoms: {len(backbone)}")
    print(f"Calculated 3D Writhe: {wr:.4f}")
    
    if abs(wr) > 2.5:
        print("Topology State: Highly entangled (Non-trivial Knotoid likely)")
    else:
        print("Topology State: Trivial / Open chain")
</code></pre>

<p><b>Dependencies:</b> This script only requires standard numerical libraries. You can execute it directly after installing numpy:</p>

<pre><code class="language-bash">pip install numpy
python calculate_topology.py
</code></pre>

<hr>

<h2 id="references">References</h2>
<ol>
  <li><b>Turaev, V. (2012).</b> <i>Knotoids.</i> Osaka Journal of Mathematics, 49(1), 195-223.</li>
  <li><b>Goundaroulis, D., Dorier, J., Benedetti, F., &amp; Stasiak, A. (2017).</b> <i>Studies of global and local entanglements of individual protein chains using the concept of knotoids.</i> Scientific Reports, 7(1), 8409.</li>
  <li><b>Sułkowska, J. I., et al. (2012).</b> <i>Conservation of complex knotting and slipknotting patterns in proteins.</i> PNAS, 109(26), E1715-E1723.</li>
</ol>
