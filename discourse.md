To evaluate the **scalability** and **performance** of the proposed algorithms under a higher client population (**N = 10**), we employed a *client replication strategy with stochastic perturbation*, given the constraint of **three unique base client datasets**.

This method involved cyclically using the three base clients as templates to generate **seven additional synthetic clients**. For each synthetic client, the complete data trajectory of its corresponding template was perturbed by adding *feature-wise Gaussian noise*, with the noise standard deviation set to **5 %** of that of the original feature:

$$
\sigma_{\text{noise}} = 0.05 \times \sigma_{\text{feature}}
$$

### Experimental Rationale

* **Objective:** Create a scenario with a larger number of agents to rigorously test the client-selection and communication-handling capabilities of the federated server.
* **Design:**

  * Three distinct **client clusters**
  * Clients within the same cluster share highly correlated data distributions
  * Differentiation introduced through local stochastic noise

This structure intentionally challenges the learning algorithms to manage groups of similar—yet non-identical—clients.


### Known Limitations

* The synthetic clients do **not** represent new, independent real-world users.
* Over-representation of the three base data distributions occurs during federated aggregation.

> **Interpretation:**
> Results from this 10-client experiment should be viewed as an evaluation of *algorithmic scalability and robustness* in a **controlled, clustered-heterogeneity** setting, **not** as an indicator of performance on a truly diverse 10-client real-world dataset.
