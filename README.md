# Standardized Ranking Correlation Coefficients

This archive contains fucntions to calculate the standardization function $g(x)$ that maps a ranking correlation coefficient $\Gamma$ to a standardized form $g(\Gamma)$ with zero expected value.
- *ranking_correlation.py* contains the function *standard_gamma_calc()*, to estimate the standardized version of the coefficient for a wide range of parameters. More details in the [`standard_gamma_calc()`](#standard_gamma_calc) section below.
- *g_estimation.py* contains utilities to replicate or extend the $g$ estimation. More details in the [Estimation of the Standardization Function](#estimation-of-the-standardization-function) section below. 
## Installation

To execute the code, please clone the archive or download its content.


-----------

## `standard_gamma_calc()`

```python
def standard_gamma_calc(coeff_type: str, weighting_scheme: str, wtype: int, aa: list, bb: list, n_0: int) -> dict:
```

**This function computes a standardized gamma value based on ranking correlation and weighting parameters.**

---

### Parameters

| Name | Type | Description |
|------|------|-------------|
| `coeff_type` | {'spearman', 'kendall'} | Type of rank correlation coefficient to use. |
| `weighting_scheme` | {'add', 'mult'} | The method used for computing weights: additive or multiplicative. |
| `wtype` | {1,2} | An integer identifier for the weight type. wtype=1 corresponds to weighting function $f(i)=1/i$, wtype=2 corresponds to $f(i)=1/(i+n_0)^2$. |
| `aa` | list of float | First ranking vector. Must be the same length as `bb`. |
| `bb` | list of float | Second ranking vector. Must be the same length as `aa`. |
| `n_0` | int | Parameter that further specifies the weigthing function. For `wtype=1` only `n_0=0` is allowed. For `wtype=2`, `n_0=0,1,2` are allowed. |

---

### Returns

A dictionary containing the following keys:
| Key | Type | Description |
|------|------|-------------|
| `standard_gamma` | float | The standardized gamma value (final output). |
| `gamma` | float | The raw gamma coefficient. |
| `gamma_avg` | float | The expected value of gamma. |
| `V` | float | The variance of gamma. |
| `V_left` | float | Left-side component of the variance of gamma. |
----

### Raises 
------ 
`ValueError`  if any parameter is outside the acceptable set of values, or if input lists are mismatched in length. 


### Range of Validity

The maximum ranking length $n_{\rm max}$ supported by the current estimation of the standardization function $g$ depends on the coefficient type, weighting scheme, and wtype.
- `coeff_type` = `'spearman'`
    - `weighting_scheme` = `'add'`
        - `wtype` = `1` $\ \to\ n_{\rm max}= 40000$
        - `wtype` = `2` $\ \to\ n_{\rm max}= \infty$
    - `weighting_scheme` = `'mult'`
        - `wtype` = `1` $\ \to\ n_{\rm max}= 40000$
        - `wtype` = `2` $\ \to\ n_{\rm max}= 40000$
- `coeff_type` = `'kendall'`
    - `weighting_scheme` = `'add'`
        - `wtype` = `1` $\ \to\ n_{\rm max}= 3000$
        - `wtype` = `2` $\ \to\ n_{\rm max}= \infty$
    - `weighting_scheme` = `'mult'`
        - `wtype` = `1` $\ \to\ n_{\rm max}= 3000$
        - `wtype` = `2` $\ \to\ n_{\rm max}= 3000$

### Examples 
-------- 
> \>\>\>  from ranking_coefficients import standard_gamma_calc <br /> 
> \>\>\>  standard_gamma_calc(coeff_type='spearman', weighting_scheme='add', wtype=2, aa=[1, 2, 3],  bb=[3, 2, 1], n_0=2 )
>
>{'standard_gamma': -1.0, <br />
> 'gamma': -1.0000000000000022,<br />
> 'gamma_avg': -0.03027621258524162,<br />
> 'V': 0.5113228395536834,<br />
> 'V_left': 0.24415735524199234}


-------- 

-------- 

-------- 


## Estimation of the Standardization Function

The file *g_estimation.py* contains utilities  to replicate or extend the $g$ estimation. Given a ranking correlation coefficient in the form
   $$
   \Gamma = \frac{\sum_{i,j}a_{ij}b_{ij}}{\sqrt{\sum_{i,j}a_{ij}^2 \sum_{i,j}b_{ij}^2}},$$

the file contains utilities to estimate the parameters of its distribution according to sampling and polynomial regression.

---

### `w_calculator_permutations(bb, n_0, aa_w, weight_type=1, weighting_scheme='add')`
Applies a weighting scheme to permutations.

**Parameters:**
- `bb` (`list[int]`): Permuted array.
- `n_0` (`int`): Reference normalization value.
- `aa_w` (`np.ndarray`): Weights of original array.
- `weight_type` (`int`, optional): Type of weight to compute. Defaults to `1`.
- `weighting_scheme` (`str`, optional): Scheme: `'add'` or `'mult'`.

**Returns:**
- `np.ndarray`: Weighted permutation array.

---

### `spearman_permutations(bb, w)`
Computes the weighted Spearman rank correlation coefficient.

**Parameters:**
- `bb` (`list[int]`): Permuted array.
- `w` (`list[float]`): Weights.

**Returns:**
- `float`: Spearman coefficient.

---

### `kendall_permutations(bb, w)`
Computes the weighted Kendall tau correlation coefficient.

**Parameters:**
- `bb` (`list[int]`): Permuted array.
- `w` (`list[float]`): Weights.

**Returns:**
- `float`: Kendall coefficient.

---

### `gamma_permutations(bb, w, coeff_type)`
Calculates gamma using the selected correlation coefficient.

**Parameters:**
- `bb` (`list[int]`): Permuted array.
- `w` (`list[float]`): Weights.
- `coeff_type` (`str`): `'spearman'` or `'kendall'`.

**Returns:**
- `float`: Gamma coefficient.

---

### `regressor(y_dic, deg, is_log, y_err_dic=None, is_train=False)`
Fits a polynomial regression to given data.

**Parameters:**
- `y_dic` (`dict[int, float]`): Mapping from `n` to output value.
- `deg` (`int`): Degree of polynomial.
- `is_log` (`bool`): Use log scale for x-axis.
- `y_err_dic` (`dict[int, float]`, optional): Error values for weights.
- `is_train` (`bool`, optional): Use training subset. Defaults to `False`.

**Returns:**
- `list[float]`: Fitted polynomial coefficients.

---

### `mse(y_dic, pars, is_log)`
Calculates Mean Squared Error of regression.

**Parameters:**
- `y_dic` (`dict[int, float]`): Ground truth.
- `pars` (`list[float]`): Polynomial coefficients.
- `is_log` (`bool`): Use log scale for x-axis.

**Returns:**
- `float`: Mean squared error.

---

### `bar_calculator(gamma_list, pars_dic=None, n=None)`
Computes mean and variance statistics for gamma.

**Parameters:**
- `gamma_list` (`list[float]`): List of gamma values.
- `pars_dic` (`dict`, optional): Regression model parameters.
- `n` (`int`, optional): Sample size.

**Returns:**
- Tuple with various statistics depending on mode:
  - Round 1: `(gamma_bar, gamma_bar_sd, '', '')`
  - Round 2: `('', '', V, V_left)`

---

### `n_samp_calc(coeff_type, n)`
Determines number of permutations to sample.

**Parameters:**
- `coeff_type` (`str`): `'spearman'` or `'kendall'`.
- `n` (`int`): Sample size.

**Returns:**
- `int`: Number of samples.

---

### `n_list_calc(coeff_type)`
Generates list of `n` values to sample.

**Parameters:**
- `coeff_type` (`str`): `'spearman'` or `'kendall'`.

**Returns:**
- `list[int]`: Values of `n` based on exponential scale.

---

### `sample_looper(...)`
Loops through `n` values and computes statistics using sampling.

**Parameters:**
- All settings for weighting and coefficients.
- `n_list` (`list[int]`): List of `n` values.
- `is_round_1` (`bool`): First or second pass.
- `pars_dic_dic` (`dict`, optional): Parameter dictionary.

**Returns:**
- Depending on round:
  - Round 1: `(gamma_dic, err_dic, {}, {})`
  - Round 2: `({}, {}, V_dic, V_left_dic)`

---

### `looper(...)`
Computes exact and sampled values of gamma and variances.

**Parameters:**
- All settings for weighting and coefficients.
- `is_round_1` (`bool`): Whether in round 1.
- `pars_dic_dic` (`dict`, optional): Regression parameters.

**Returns:**
- Round 1: `(gamma_dic_exact, gamma_dic_sample, err_dic_sample, '', '')`
- Round 2: `('', '', '', V_dic_exact, V_left_dic_exact, V_dic_sample, V_left_dic_sample)`

---

### `optimal_deg_calc(mse_dic_n_0)`
Selects optimal polynomial degree based on MSE ratios.

**Parameters:**
- `mse_dic_n_0` (`dict[int, float]`): Degree to MSE mapping.

**Returns:**
- `int`: Optimal polynomial degree.

---

### `regressor_final(dic_dic, opt_deg_dic, coeff_type, weighting_scheme, wtype, n_0, y_type='gamma')`
Performs final regression using optimal parameters.

**Parameters:**
- `dic_dic` (`dict`): Data dictionary.
- `opt_deg_dic` (`dict`): Optimal degree dictionary.
- `coeff_type` (`str`): `'spearman'` or `'kendall'`.
- `weighting_scheme` (`str`): Weighting method.
- `wtype` (`int`): Weight type.
- `n_0` (`int`): Reference normalization.
- `y_type` (`str`, optional): Output type. Defaults to `'gamma'`.

**Returns:**
- `dict`: Regression model with `'is_log'` and `'pars'`.

---

### `opt_dec_dic_fill(opt_deg_dic, dic_dic, max_deg, y_type='gamma')`
Fills in the optimal degree dictionary based on MSE comparison.

**Parameters:**
- `opt_deg_dic` (`dict`): Optimal degrees to be updated.
- `dic_dic` (`dict`): Data dictionary.
- `max_deg` (`int`): Maximum degree to try.
- `y_type` (`str`, optional): Statistic to fit. Default is `'gamma'`.

**Returns:**
- `dict`: Updated `opt_deg_dic`.


