====Graph Generation====

The tool employed for skull stripping/brain extraction is [[http://fsl.fmrib.ox.ac.uk/fsl/fslwiki/BET/UserGuide|FSL's BET]]. This tool was developed by [[http://onlinelibrary.wiley.com/doi/10.1002/hbm.10062/epdf|S.M. Smith]] in 2002.

Let $\textbf{x} = (x_i, ... , x_n)$.\\
Let $\textbf{y} = (y_i, ... , y_m)$, $y_i^{(q)} \in \mathscr{Y} =[n_W] \times [n_L] \times [n_H]$.\\
Let $t_j = F_I(j)$ be cdf of $I(\textbf{x})$. \\
Let $\Theta(x < \tau) = \begin{cases} 0 & x \geq \tau \\ 1 & x < \tau \\ \end{cases}$ \\
Let $\mathcal{N}(y_i) = \{ y_j : y_i \sim y_j\}, n_b=\lvert \mathcal{N}(y_i) \rvert$.\\
Let $\Phi_i^{(q)} = \sum\limits_{j,k \in \mathscr{N}(y_i)}^{n_b} (y_i - y_j) \times (y_i - y_k)$. \\
Let $l^{(q)} = \frac{1}{m} \sum\limits_{i}^m \frac{1}{n_b} \sum\limits_{j \in \mathscr{N}(y_i)}^{n_b} \lvert y_i - y_j \rvert$.

**Constants**: Minimum and maximum curvature $r_{min}$, $r_{max}$, radial search distance $d_1$, $d_2$

**Inputs**: $x_i \in \mathscr{X}= [n_W] \times [n_L] \times [n_H]$, fractional learning rate $b \in (0,1)$, stopping threshold $\epsilon$, image $I(\textbf{x}) \in \mathscr{I} \subset \mathbb{R}^{[n_W] \times [n_L] \times [n_H]}$

**Outputs**: brain $B(\textbf{x} \in \mathscr{I})$, brain mask $M(\textbf{x} \in \mathscr{I})$

  - Compute robust intensity extrema $t_2$, $t_{98}$
  - Estimate brain/background threshold $\tau = 0.1(t_{98} - t_{2})$
  - Estimate centroid $c = \frac{1}{n_\tau} \sum\limits_{i=1}^n x_i \Theta(I(x_i) > \tau) I(x_i)$, where $n_\tau = \sum\limits_{i=1}^n \Theta(I(x_i) > \tau)$
  - Estimate $r = (0.75 n_\tau$ / $ \pi)^{1/3}$
  - Compute median, $t_{50}$
  - Initialize $\textbf{y}$ as vertices of a spherical tessellated mesh centered on $c$ with radius $r/2$
  - Initialize $\lvert u_i^{(1)} \rvert = \infty$, $\forall i=[n]$
  - **while (**$\lvert u_i^{(q)} \rvert \geq \epsilon$**)**
    - Compute local surface normal, $\hat{n}_i^{(q)} = \Phi_i^{(q)}$ / $n_b^{(q)} \lvert \Phi_i^{(q)} \rvert$
    - Compute mean local vertex position, $v_i^{(q)} = \frac{1}{n_b}\sum\limits_{j\in \mathscr{N}(y_i)}^{n_b} y_j$
    - Compute displacement vector for each vertex, $s_i^{(q)} = y_i - v_i^{(q)}$
    - Compute normal, $\eta_i^{(q)} = (s_i^{(q)} \cdot \hat{n}_i^{(q)}) \hat{n}_i^{(q)}$, and tangential, $\gamma_i^{(q)} = s_i^{(q)} - \eta_i^{(q)}$, and component vectors
    - Compute update vector $u_i^{(q)}$ as in equation $(1)$ below
    - Update $y_i^{(q+1)} = y_i^{(q)} + u_i^{(q)}$
  - **end while**
  - Generate mesh, $m(\textbf{y}) \in \mathscr{M} \subset \mathscr{Y}$
  - Compute $B(\textbf{x}) = \begin{cases} 0 & x_i \notin \mathscr{M} \\ I(\textbf{x}) & x_i \in \mathscr{M} \\ \end{cases}$
  - Compute $M(\textbf{x}) = \begin{cases} 0 & x_i \notin \mathscr{M} \\ 1 & x_i \in \mathscr{M} \\ \end{cases}$
