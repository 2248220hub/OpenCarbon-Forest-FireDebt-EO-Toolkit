# Contributing

Thank you for helping make fire-emission analysis more open.

**Ways to help**

- **Run it on another fire** and open an issue with your Block 0 settings and results register — case studies are the most useful contribution.
- **Report a problem** — include the block number, the full error message and your Block 0 values (leave out your project ID).
- **Improve a method** — new emission-factor tables for other biomes, VIIRS fire radiative power, GEDI or Sentinel-1 blocks.

**Ground rules**

1. Keep everything runnable in Colab on free Earth Engine access.
2. Every new number goes through `rec()` or `rec_model()` so it reaches the register.
3. Keep site-specific values in Block 0.
4. Cite the source of any constant you add, in the code comment and in `docs/03_METHODS.md`.
