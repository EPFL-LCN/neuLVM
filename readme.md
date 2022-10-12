[![DOI](https://zenodo.org/badge/550245001.svg)](https://zenodo.org/badge/latestdoi/550245001)

This is python code collection for the publication:

Mesoscopic modeling of hidden spiking neurons 

Contact: shuqi.wang@epfl.ch

## Installation: 

```
conda create -n neuLVM python=3.8
conda activate neuLVM
conda install pip
pip install -r requirements.txt
```

## Usage:
The main results (multi-pops winner-take-all experiment) of the publication can be replicated with the main script `train_meso.py`.

One example training dataset (11 s-long) is saved in `dataset/3pop_noinput_small`. To train a neuLVM, one can run the bash script `3pop_neuLVM.sh`, in which we do five runs of `train_meso.py` with different initialized parameters. The results will be saved in `result/3pop example/data11/train{i}/`, including:
```
E{EMstep}_est_param: to retrieve the inferred pop. activity
minimizor_result_{EMstep}: to retrieve the fitted model/ parameters

E_{EMstep}_{GDstep}.png: visualization of the initial estimate of pop. activity
init_param: initialization of the model/ parameters
Eloss.png
```


### What happened in `train_meso.py`
This code fits a winner-take-all circuit of three homegenenous populations of LIF neurons to the spike trains of nine recorded neurons. This code was used to produce the Figure 4 from our paper. For simplicity, the correct partitioning of the nine recorded neurons into three groups is given since it can be reliably obtained (more details are available in the paper). The fitting procedures (the function `train` in the file `train_meso.py`) contains 3 steps: initialization, E and M which we describe below.

\[Data-driven initialization\]: Since the sum over the observed neurons from population already provides a rough estimate of the latent population activity, the E-Step of the first iteration is replaced by an empirical estimation of the population activity from the observed spike trains (the function `init_model` in the file `src/helper.py`). And the EM iterations will start with the M-step. In the Figure below we show the population activity resulting from the initialization.
![initial estimate of pop. act.](/fig/init_Z.png "initial estimate of pop. act.")

\[M-step $\widehat{\Theta}^{n} = \operatorname{argmax}_\Theta \log p(\mathbf{y^o},\widehat{\mathbf n}^{n}| \Theta)$\]: Parameters (since there are not so many) are estimated with `scipy.optimize.minimize` (the function `scipy_Mstep` in the file `train_meso.py`). M-step optimizes Eq (4b) of the paper: 

<!-- ![likelihood equation](/fig/llh_eq.png "likelihood equation") -->
<p align="center">
  <img src="/fig/llh_eq.png" />
</p>

the probability (part a) of the observed spikes given the past observed spike activity and the past population activity (left part) is computed by the function `fast_lossy` in the file `src/fast_np.py`; and the probability (part b) of the population activity given the past population activity (right part) is computed by the function `fast_lossZ` in the file `src/fast_np.py`.

\[E-step $\widehat{\mathbf n}^{n} = \operatorname{argmax}_\mathbf n \log p(\mathbf{y^o},\mathbf n| \widehat{\Theta}^{n-1})$\]: Pop. act. are estimated with Adam algorithm (the function `tf_EGDstep` in the file `train_meso.py`). Losses are computed in the same way as above, only in the language of TensorFlow (`src/LIFmesoCell.py`). 

In the Figure below we show the inferred population activity after EM iterations.
![inferred pop. act.](/fig/inferred_Z.png "inferred pop. act.")


### Free simulations
To simulate trials with trained model/parameters, one can run the python file `simulate_with_trainedparams.py`. In the Figure below we show the free simulations of the trained model.
![Free simulations](/fig/sim_Z.png "Free simulations")



