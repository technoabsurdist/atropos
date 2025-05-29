# ⏱️ Baseline Timing Results


This folder contains a set of baseline timing results for the KernelBench problems.

Since KernelBench measures the speedup between Runtime(refernece architecture) and Runtime(LLM-generated architecture), it is important to measure the baseline reference module runtime.

We have provided a set of baseline results for the KernelBench problems on a variety of hardware as well as various PyTorch configurations.
All baseline are ran with PyTorch `2.5.0+cu124` and CUDA `12.4`.

For timing, we measure wall clock time. We warm up 3 times and collect runtime statistics for 100 trials.

### Run on your own cluster
Since your cluster might be different from ours (different GPU, different power setting, etc.), you can create the baseline results on your own cluster.
Take a look at `python3 scripts/generate_baseline_time.py` to see how to set and run timing results.

### Run on modal
To gather baseline on modal, take a look at `python3 scripts/generate_baseline_time_modal.py` to see how we are doing it.

### Hardware

We profile baseline KernelBench problem times on a variety of hardware, across different GPU Generations.

| Provider | GPU Type | Memory | Power | GPU Architecture |
|----------|----------|---------|--------|--------------|
| Stanford matx3 | NVIDIA L40S | 48 GB | 300W | Ada |
| Together.ai | NVIDIA H100 | 80 GB | 700W | Hopper |
| Modal | NVIDIA L40S | 48 GB | 350W | Ada |
| Modal | NVIDIA H100 | 85 GB | 700W | Hopper |
| Modal | NVIDIA A100 | 42 GB | 400W | Ampere |
| Modal | NVIDIA L4 | 24 GB | 72W | Ada |
| Modal | NVIDIA T4 | 16 GB | 70W | Turing |
| Modal | NVIDIA A10G | 24 GB | 300W | Ampere |

Check the `timing` directory which have the timing results for each of the above hardware.


### Timing Configurations

We are interested in Torch eager execution as well as Torch Compile with various backends and modes.

| Configuration | Backend | Mode | Description |
|--------------|---------|------|-------------|
| Torch (Eager) | - | - | Standard PyTorch eager execution |
| Torch Compile | inductor | default | Default torch.compile behavior |
| Torch Compile | inductor | reduce-overhead | Optimized for reduced overhead |
| Torch Compile | inductor | max-autotune | Maximum autotuning enabled |
| Torch Compile | inductor | max-autotune-no-cudagraphs | Maximum autotuning without CUDA graphs |
| Torch Compile | cudagraphs | - | CUDA graphs with AOT Autograd |


Learn more about Torch Compile [backends](https://pytorch.org/docs/stable/torch.compiler.html) and [modes](https://pytorch.org/docs/main/generated/torch.compile.html).


### Acknowledgements

Thank you to [@PaliC](https://github.com/PaliC) from the PyTorch team for the exerptise on various Torch Configurations.

Thanks to [Modal](https://modal.com/) for sponsoring compute credits for us to collect runtime baseline on a vareity range of NVIDIA GPUs.