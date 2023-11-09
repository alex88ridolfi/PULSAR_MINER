# PULSAR_MINER

___“Edit the configuration file, input your observation to search, and you are ready to go!”___




PULSAR_MINER is a simple, easy-to-use, pulsar searching pipeline.\
Its goal is to to make pulsar searching as fast and straightforward as possible.

In essence, it is a convenient Python3 wrapper of PRESTO, one of the most popular and widely used pulsar searching package.

PULSAR_MINER runs all the PRESTO routines in a consistent, efficient, and fully automated way. It is also resilient to interruptions, as it can cleverly  resume from the point where it stopped, so as to not waste any your resources and time.

## Requirements

At minimum, PULSAR_MINER requires:
- [*crucial*] Python 3.8 or newer with the [Numpy](https://numpy.org/) library
- [*crucial*] [PRESTO 3.0](https://github.com/scottransom/presto) or newer
- [*crucial*] the PRESTO Python3 libraries


 If you want to take advantage of the GPU acceleration (highly recommended) for the accelearation-search part, you also need:


- [*optional*] A CUDA-compatible GPU (i.e. an NVIDIA GPU) and Jintao Luo's [PRESTO2_ON_GPU](https://github.com/jintaoluo/presto2_on_gpu)

**Note 1**: Jintao Luo's PRESTO2_ON_GPU has not been updated in many years and is not compatible with CUDA 11.6 or later. Alessandro Ridolfi has a [forked version](https://github.com/alex88ridolfi/presto2_on_gpu) that works with CUDA 11.6 to 11.8, and that should also be a tiny bit easier to install. Compatibility is, however, fully broken from CUDA 12 onwards.




## Installation

PULSAR_MINER itself does *not* require any installation procedure. 

1) Once you made sure that you have Python 3.8+ and PRESTO 3.0+ installed, you can simply get the latest version of PULSAR_MINER by downloading the git repository: 

   `git clone https://github.com/alex88ridolfi/PULSAR_MINER`


2) If you want to have PULSAR_MINER and its auxiliary tools always at hand, add the PULSAR_MINER directory to your `~/.bashrc` file:

   `echo "export PATH=/path/to/PULSAR_MINER:\${PATH}" >> ~/.bashrc`

   where, of course, `/path/to/PULSAR_MINER` must be replaced with the actual path of the downloaded repository.

3) Log-out and re-login to your terminal to make the changes effective.

#### GPU Acceleration

To enable the GPU acceleration in PULSAR_MINER, 

4.  Install CUDA version 11.8 or older on your system

    **Note**: Remember that, even if your system already has CUDA 12.0 or newer installed, you can always install an older version of CUDA alongside your main one. See installation notes on Alessandro Ridolfi's [forked version of PRESTO2_ON_GPU](https://github.com/alex88ridolfi/presto2_on_gpu)


5.  Install PRESTO2_ON_GPU with CUDA enabled.

    - If you are using CUDA version <= 11.5.2,  install Jintao Luo's  [original version of PRESTO2_ON_GPU](https://github.com/jintaoluo/presto2_on_gpu) 
    - If you are using CUDA version 11.6 to 11.8, install Alessandro Ridolfi's [forked version of PRESTO2_ON_GPU](https://github.com/alex88ridolfi/presto2_on_gpu), which should also be a tiny bit easier to install.
    

6. Make sure to have the `PRESTO2_ON_GPU` environment variable with the path to your PRESTO2_ON_GPU installation
 
   `echo "export PRESTO2_ON_GPU=/path/to/presto2_on_gpu" >> ~/.bashrc `


## Computational considerations

PULSAR_MINER has been designed to work most efficiently on a single workstation. It can greatly benefit from the presence of:
 - Several CPU cores/threads (for the de-dispersion, jerk search, and folding)
 - SSD/NVMe storage (to minimize I/O bottlenecks)
 - NVIDIA GPU(s) (for the acceleration search, requires  [PRESTO2_ON_GPU](https://github.com/jintaoluo/presto2_on_gpu) installed)

If you would like to use PULSAR_MINER on a large multi-node computing clusters based on the SLURM workload manager, I recommend you to use Vishnu Balakrishnan's [SLURM PULSAR_MINER](https://github.com/vishnubk/SLURM_PULSARMINER).


## Basic usage

To successfully use PULSAR_MINER, you need:

- A search-mode observation file in a format compatible with PRESTO (tipically a `.fil` filterbank file or a `.fits`/`.sf` psrfits file).

- A `.cfg` configuration file containing all the relevant search parameters. A template of such a file can be generated using the `-init_default` option.



Assuming to have an observation file called `NGC6752_MeerKAT_0001.fits`:

1. Generate a configuration file template with the command:

   ```
      pulsar_miner -init_default NGC6752_MeerKAT_0001.fits
   ```

   The code will create a few folders and files, among which a template configuration file named after the current working directory (`NGC6752.cfg` in this example). The configuration file contains all the relevant parameters for the search itself, with a short description of their meaning. The code also tries to autonomously give the correct values to several of these parameters (e.g. your PRESTO installation directory, the number of CPU cores available etc.) by reading some environment variables and by getting some system information. __However, it is highly recommended that you cross-check these values.__
   Also, the majority of the parameters (such as the DM range to search, the acceleration search parameters etc.) are set to some generic default values. Obsvioulsy you may want to change them, since they will likely not be the optimal ones for your own search.

   The parameters in the configuration file can be changed using a simple text editor, sucs as emacs.
   ```
      emacs NGC6752.cfg
   ```
   
   If you expect to find some previously known pulsars in the data (such as for the case of most globular cluster observations), you can place their `.par` files in the `known_pulsars` folder, so as to make PULSAR_MINER aware of them.

2. Once you are happy with the parameters of the configuration file, run the pipeline through the command:
   ```
      pulsar_miner -config NGC6752.cfg -obs NGC6752_MeerKAT_0001.fits
   ```
   and hopefully you can just wait for it to finish!

3. If everything went well, you should end up with five folders in your working directory:
   ```
      $ ls
      01_RFIFIND        04_SIFTING           known_pulsars 
      02_BIRDIES        05_FOLDING           NGC6752_MeerKAT_0001.fits       
      03_DEDISPERSION   common_birdies.txt   NGC6752.cfg
   ```

   To look for the resulting candidates, go into `05_FOLDING/NGC6752_MeerKAT_0001`. Here you can find all the folding plots. Hopefully some new pulsars will appear among them! :-)



## References
If you effectively used PULSAR_MINER for your work, I would appreciate if you could cite:

 - The [PULSAR_MINER  GitHub repository](https://github.com/alex88ridolfi/PULSAR_MINER), as well as the [Ridolfi et al. (2021)](https://ui.adsabs.harvard.edu/abs/2021MNRAS.504.1407R/abstract) paper, where the pipeline was first used and described.

- The [PRESTO GitHub repository](https://github.com/scottransom/presto), as well as the appropriate references as indicated in [its README.md file](https://github.com/scottransom/presto/blob/master/README.md). 




## Acknowledgements
Over the years, PULSAR_MINER has been used by many people, and their feedback has been extremely valuable.
Big thanks to Tasha Gautam, Laila Vleeschower Calas, Vishnu Balakrishnan, Weiwei Chen, Rouhin Nag, Federico Abbate, Marta Burgay, for using the pipeline, reporting bugs, and suggesting improvements.
A special thanks to Tasha Gautam for adding the initial support to jerk search.



