# <center>GROMACS</center>

---------

GROMACS是一种分子动力学应用程序，可以模拟具有数百至数百万个粒子的系统的牛顿运动方程。GROMACS旨在模拟具有许多复杂键合相互作用的生化分子，例如蛋白质，脂质和核酸。有关GROMACS的更多信息，请访问[http://www.gromacs.org/](http://www.gromacs.org/)。

## 模块加载方法

可以使用module直接加载GROMACS应用，提供了不同的版本：

| 版本 | 加载方式 |
| ---- | ------ |
| 2018.2(gcc)   | module load gromacs/2019.2-gcc-8.3.0-openmpi |
| 2019.2(intel) | module load gromacs/2019.4-intel-19.0.4-impi |

## 作业脚本示例

使用intel编译的GROMACS运行单节点作业脚本示例gromacs_cpu_intel.slurm如下：

```bash
#!/bin/bash
#SBATCH -J gromacs_cpu_test
#SBATCH -p cpu
#SBATCH -o %j.out
#SBATCH -e %j.err
#SBATCH -n 40
#SBATCH --ntasks-per-node=40
#SBATCH --exclusive

module purge
module load gromacs/2019.4-intel-19.0.4-impi

ulimit -s unlimited
ulimit -l unlimited

export I_MPI_PMI_LIBRARY=/usr/lib64/libpmi.so
export I_MPI_FABRICS=shm:dapl

srun gmx_mpi mdrun -deffnm test -ntomp 1
```

并使用如下指令提交：

```bash
$ sbatch gromacs_cpu_intel.slurm
```

使用gcc编译的GROMACS运行单节点作业脚本示例gromacs_cpu_gnu.slurm如下：

```bash
#!/bin/bash
#SBATCH -J gromacs_cpu_test
#SBATCH -p cpu
#SBATCH -o %j.out
#SBATCH -e %j.err
#SBATCH -n 40
#SBATCH --ntasks-per-node=40
#SBATCH --exclusive

module purge
module load gromacs/2019.2-gcc-8.3.0-openmpi

ulimit -s unlimited
ulimit -l unlimited

srun --mpi=pmi2 gmx_mpi mdrun -deffnm test -ntomp 1
```

并使用如下指令提交：

```bash
$ sbatch gromacs_cpu_gnu.slurm
```


## 使用singularity容器中的GROMACS

集群中已经预置了NVIDIA GPU CLOUD提供的优化镜像，通过调用该镜像即可运行GROMACS作业，无需单独安装，目前版本为2018.2。该容器文件位于/lustre/share/img/gromacs-2018.2.simg

## 使用singularity容器提交GROMACS作业

如需使用GPU运行GROMACS，需要指定使用dgx2分区。以下是基于Singularity的作业脚本gromacs_gpu_singularity.slurm示例：

```bash
#!/bin/bash
#SBATCH -J gromacs_gpu_test
#SBATCH -p dgx2
#SBATCH -o %j.out
#SBATCH -e %j.err
#SBATCH -n 6
#SBATCH --ntasks-per-node=6
#SBATCH --gres=gpu:1

IMAGE_PATH=/lustre/share/img/gromacs-2018.2.simg

ulimit -s unlimited
ulimit -l unlimited

singularity run --nv $IMAGE_PATH gmx mdrun -deffnm benchmark -ntmpi 6 -ntomp 1
```

并使用如下指令提交：

```bash
$ sbatch gromacs_gpu_singularity.slurm
```

## 性能评测

测试使用了GROMACS提供的Benchmark算例和两位用户提供的算例进行了CPU和GPU的性能进行对比。其中cpu测试使用单节点40核心，dgx2测试分配1块gpu并配比6核心。

| (ns/day) | CPU (2019.2-gcc) | CPU (2019.4-intel) | dgx2 (Singularity) | dgx2 (2019.2-gcc) |
| ---- | ------ | ------ | ------ | ------ |
| Benchmark | 49.281 | 64.800 | 117.593 | 124.219 |
| 算例1      | 33.916 | 35.143 |  14.902 |  17.171 |
| 算例2      | 16.771 | 13.781 |   5.436 |   6.643 |

｜ 性能数据供用户选择参考。

## 参考文献

- [gromacs官方网站](http://www.gromacs.org/)
- [NVIDIA GPU CLOUD](ngc.nvidia.com)
- [Singularity文档](https://sylabs.io/guides/3.5/user-guide/)