# TersoffForge

**TersoffForge** is a Python-based utility to generate LAMMPS-compatible `.tersoff` potential files from a structured JSON configuration. This tool automates the generation of all $N^3$ triplets and applies custom mixing rules and parameter zeroing as required for complex multi-element systems.

## Author Information

- **Author**: Soheil Solhjoo - s.solhjoo@rug.nl
- **Date**: March 1, 2026
- **Affiliation**: ESD-ENTEG-FSE-University of Groningen

## Features

- **Automated Triplet Generation**: Generates all $N^3$ combinations (e.g., 8 triplets for 2 elements, 27 for 3 elements, 64 for 4 elements).
- **Custom Mixing Rules**:
  - **Arithmetic Mean**: Used for exponents ($\lambda_1, \lambda_2$).
  - **Geometric Mean**: Used for energy and cutoff parameters ($A, B, R, S$).
- **Bond-Order Scaling**: Applies $\chi$ (Chi) scaling specifically to the $B$ parameter based on the bond type.
- **Parameter Zeroing**: Automatically zeroes $n, \beta, \lambda_1, \lambda_2,$ and $A$ for triplets where the neighbor atoms are different ($j \neq k$).
- **LAMMPS Conversion**: Converts $R$ and $S$ (cutoffs) to the LAMMPS $R$ and $D$ parameters.
- **Reference Support**: Includes a formatted reference in the generated file header.

## Usage

1. **Prepare your `params.json`**: Define your elements, their base parameters, and the mixing coefficients ($\chi$).
2. **Run the script**:
   ```bash
   python tersoff_builder.py params.json Si3N4.tersoff
   ```

## How It Works: Key Concepts

### 1. Generating Subsets of Potentials
The generation of triplets is **strictly driven by the `elements` list** in your JSON. The `parameters` dictionary acts as a library of all potential atoms. 
- If you have Si, N, and H in `parameters` but only want a Silicon potential, set `"elements": ["Si"]`. The tool will generate exactly 1 triplet (`Si-Si-Si`) and ignore the rest.
- To generate the full Si-N-H system, set `"elements": ["Si", "N", "H"]`.

### 2. Symmetry in `mix-Chi`
The tool is designed to be user-friendly and order-independent regarding bond scaling:
- **Automatic Fallback**: For any mixed bond $i-j$, the code first looks for a key like `"Si-N"`. If it isn't found, it automatically checks for `"N-Si"`.
- **Defaults**: If neither permutation is found in the `mix-Chi` dictionary, the tool assumes a default scaling of **1.0**.
- **Flexibility**: You only need to define the mixing parameter once (e.g., `"Si-N": 0.65`) for it to be applied correctly to all relevant triplets (e.g., `Si-N-N`, `N-Si-Si`, etc.).

### 3. Technical Assumptions & Constraints
- **Central Atom Dominance**: For any triplet $i, j, k$, the angular parameters ($c, d, h$) and bond-order exponents ($m, \gamma, \lambda_3$) are strictly taken from the central atom $i$.
- **Mixed Bond Order**: For mixed $i-j$ bonds, the parameters $n$ and $\beta$ are taken from the central atom $i$.
- **Zeroing Logic**: The script follows the convention that $n, \beta, \lambda_1, \lambda_2,$ and $A$ are **zeroed** whenever the two neighbor atoms are different ($j \neq k$). This is specific to certain Tersoff variants like the Si-N potential by Hammer & Nørskov.

## LAMMPS Documentation

For more information on the Tersoff potential format and parameters, see the [LAMMPS pair_style tersoff documentation](https://docs.lammps.org/pair_tersoff.html).

## JSON Input Format

```json
{
  "reference": "Mota et al., Si3N4 Tersoff potential parameters: https://link.aps.org/doi/10.1103/PhysRevB.58.8323",
  "elements": ["Si", "N"],
  "parameters": {
    "Si": {
      "m": 3.0, "gamma": 1.0, "lambda3": 0.0,
      "c": 100390, "d": 16.217, "h": -0.59825,
      "n": 0.78734, "beta": 1.1e-6,
      "lambda2": 1.73222, "B": 471.18,
      "R": 2.70, "S": 3.00,
      "lambda1": 2.4799, "A": 1830.8
    },
    "N": { ... }
  },
  "mix-Chi": {
    "Si-Si": 1.0,
    "N-N": 0.0,
    "Si-N": 0.65
  }
}
```
