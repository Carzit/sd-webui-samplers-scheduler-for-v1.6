# sd-webui-samplers-scheduler Seniorious for v1.6.x

## Introduction
A samplers scheduler which can apply different sampler in diffrent generation steps,  

I hope it will be helpful to achieve a balance between generation speed and image quality.

Our paper on arXiv: [Sampler Scheduler for Diffusion Models](https://arxiv.org/abs/2311.06845)  

## Requirement
SD WebUI Version >= 1.6.0  

(For version <= 1.5.2, please use https://github.com/Carzit/sd-webui-samplers-scheduler)

## How to use
This repository is a extension for sd webui. Just place it in the `extension` folder!😉  

Choose the Sampler `Seniorious` or `Seniorious Karras` to enable the samplers scheduler.  

*`Seniorious` uses nomal noise scheduler and `Seniorious Karras` uses the noise scheduler  
recommended in [Elucidating the Design Space of Diffusion-Based Generative Models](https://arxiv.org/abs/2206.00364) (Karras et al., 2022)  

<img src="https://github.com/Carzit/sd-webui-samplers-scheduler-for-v1.6/blob/main/images/samplers_ui.png" width="500px">

This samplers scheduler provides 8 sampler units (Sampler 1-8). 

You can choose what kind of sampler used in each unit(choose `None` to unable), and the inference steps for each unit.  

The image generation process will follow the configurations of these 8 units in sequence. 

<img src="https://github.com/Carzit/sd-webui-samplers-scheduler-for-v1.6/blob/main/images/scheduler_ui.png" width="500px">

**Attention: The total steps should be equal to the sum of the steps in every unit!**  
Open the `Check` accordion and press the `Check` button to check total steps.   
`Total steps in Seniorious` shows the sum of steps in your Sampler Scheduler Settings and `Total steps required` shows the steps you set in webui. They must be equal.  

<img src="https://github.com/Carzit/sd-webui-samplers-scheduler-for-v1.6/blob/main/images/check_ui.png" width="500px">

If you use Sampler Scheduler Seniorious, the `Sampler Scheduler Config` will be added to the `png_info`.

<img src="https://github.com/Carzit/sd-webui-samplers-scheduler-for-v1.6/blob/main/images/png_info.png" width="500px">

## Available Samplers
14 kinds of mainstream samplers in [k-diffusion](https://github.com/crowsonkb/k-diffusion) are available:  

- `Euler`
- `Euler a`
- `Heun`
- `Heun++`
- `LMS`
- `DPM2`
- `DPM2 a`
- `DPM++ 2S a`
- `DPM++ SDE`
- `DPM++ 2M`
- `DPM++ 2M SDE`
- `DPM++ 3M SDE`
- `Restart` 
- `LCM`

You can also choose `Skip` to skip certain steps.  

`Heun++` is my [test version](https://github.com/Carzit/sd-webui-sampler-heunpp).  
`Restart` is a new sampler in SD WebUI since version 1.6, using the recommended hyperparameters recommended in [Implements restart sampling in Restart Sampling for Improving Generative Processes](https://arxiv.org/abs/2306.14878).  
`LCM` is a new sampler used in latent consistency model(LCM).  

## FID Result
I calculate the FID Score based on [EDM](https://github.com/NVlabs/edm).

config:  
- seeds=0-49999  
- network=https://nvlabs-fi-cdn.nvidia.com/edm/pretrained/edm-cifar10-32x32-cond-vp.pkl  
- NFE=30
  
| Sampler[steps] | Type | FID | 
| :-----: | :----: | :----: |
| Euler[30] | Global ODE | 4.20923 | 
| Heun[16] | Global ODE | 8.8082 |
| DPM++2M[30] | Global ODE | 2.06226 |
| DPM2[16] | Global ODE | 1.94304 |
| Euler a[30] | Global SDE | 14.5688 |
| DPM2 a[16] | Global SDE | 3.58586 |
| DPM++ 2S a[16] | Global SDE | 9.74262 |
| DPM++ SDE[16] | Global SDE | 17.807 |
| Heun[10] - DPM2[5] | Sampler Scheduler ODE | 1.86846 |
| DPM2 a[10] - DPM2[5] | Sampler Scheduler SDE+ODE | 1.84535 |

Discretely scheduling different samplers during the sampling process has proven to be effective at a practical level.
  
## Examples

### ODE Samplers

![](https://github.com/Carzit/sd-webui-samplers-scheduler/blob/main/images/example2.png)  

BRISQUE Score: 
| Sampler | BRISQUE |
| :-----:| :----: |
| Euler | 23.3771 |
| DPM++ 2M | 27.5705 |
| DPM++ 2M Karras | 24.8244 |
| Heun | 21.4943 |
| DPM 2 | 22.1520 |
| Seniorious | 21.3698 |
| Seniorious Karras | 21.6955 |  

Shared Parameters: 
- Prompt: best quality, masterpiece, 1girl, solo, standing, sky, portrait, looking down, floating light particles, sunshine, cloud, depth of field, field, wide shot
- Negative prompt: badhandv4, By bad artist -neg, EasyNegative, NegfeetV2, ng_deepnegative_v1_75t, verybadimagenegative_v1.3
- Steps: 30
- CFG scale: 7
- Seed: 2307198650
- Size: 512x768,
- Model: anything-v5-PrtRE  

Sampler Scheduler Parameters:  
| Unit | Sampler | Steps |
| :-----:| :----: | :----: |
| Sampler1 | Heun | 10 steps |
| Sampler2 | DPM++2M | 10 steps |
| Sampler3 | Euler | 10 steps |  

*Seniorious and Seniorious Karras use the same parameters in this example.

### Put ODE and SDE Samplers Together
![](https://github.com/Carzit/sd-webui-samplers-scheduler/blob/main/images/example3.png)  

Sampler Scheduler Parameters:  
| Unit | Sampler | Steps |
| :-----:| :----: | :----: |
| Sampler1 | DPM2 a | 20 steps |
| Sampler2 | DPM2 | 10 steps |

I recommend to use SDE in the early sampling steps and ODE in the later sampling steps to solve the inherent problems previously caused by using either singly.

### About LCM Sampler
Like other implemented sampling algorithms, I added the LCM sampler as a sampler option to our Sampler Scheduler.  
But obviously the iteration formula of LCM sampler $x_{i+1} = D_\theta(x_{i}) + \sigma_{i+1}	\epsilon_i$ is different from those of traditional LDM samplers.   
Ignoring its coupling with the LCM model and using it rashly as a plug-in sampler for LDM may cause some problems, such as blurring at the edge of the image.

<img src="https://github.com/Carzit/sd-webui-samplers-scheduler-for-v1.6/blob/main/images/example_lcm1.png" width="200px"><img src="https://github.com/Carzit/sd-webui-samplers-scheduler-for-v1.6/blob/main/images/example_lcm2.png" width="200px">


Moreover, based on some of my experience, I recommend using LCM sampler at later sampling steps rather than at the beginning.  
In figures above, with other parameters fixed, the left uses 12-step `Euler a` + 8-step `LCM` while the right uses 8-step `LCM` + 12-step `Euler a`.


## More
The idea of this extension was inspired by Seniorious, a Carillon composed of different talismans.  

Different talismans correspond in sequence to make Seniorious a powerful weapon, and so do the samplers in the samplers scheduler.  

In the end, many thanks to Chtholly Nota Seniorious, the happiest girl in the world.
