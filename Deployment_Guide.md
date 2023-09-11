# End-to-End Computer Vision Bootcamp

The **End-to-End Computer Vision Bootcamp** is designed from a real-world perspective and follows the data processing, development, and deployment pipeline paradigm using a variety of tools. Through hands-on exercises, attendees will learn the fundamentals of preprocessing custom images, speeding the development process using transfer learning for model training, and deployment of trained models for fast and scalable AI in production.


## Deploying the Labs

### Prerequisites

To run this tutorial you will need a Laptop/Workstation/DGX machine with NVIDIA GPU.

- Install the latest [Docker](https://docs.docker.com/engine/install/) or [Singularity](https://sylabs.io/docs/).
    - Once you have installed **docker**, follow the [post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/) to ensure that docker can be run without `sudo`.

- Get an NGC account and API key:

    - Go to the [NGC](https://ngc.nvidia.com/) website and click on `Register for NGC`.
    - Click on the `Continue` button where `NVIDIA Account (Use existing or create a new NVIDIA account)` is written.
    - Fill in the required information and register, then proceed to log in with your new account credentials.
    - In the top right corner, click on your username and select `Setup` in the dropdown menu.
    - Proceed and click on the `Get API Key` button.
    - Next, you will find a `Generate API Key` button in the upper right corner. After clicking on this button, a dialog box should appear and you have to click on the `Confirm` button.
    - Finally, copy the generated API key and username and save them somewhere on your local system.

### Tested environment

All Labs were tested and is set to run on a DGX machine equipped with an Ampere A100 GPU. It was also tested using a workstation equipped with an NVIDIA RTX A3000 GPU with 6GB of VRAM, reducing all the batch sizes to 8 during training. 
The results may vary when using different hardware and some hyperparameters may not be ideal for fully taking advantage of the graphic card.


### Deploying with container

This material can be deployed with either Docker or Singularity container, refer to the respective sections for the instructions.

#### Running Docker Container

##### Lab 1 & 2

**Install dependencies**

1. Create a new `conda` environment using `miniconda`:

    - Install `Miniconda` by following the [official instructions](https://conda.io/projects/conda/en/latest/user-guide/install/).
    - Once you have installed `miniconda`, create a new environment by setting the Python version to 3.6:
    
        `conda create -n launcher python=3.6`
    
    - Activate the `conda` environment that you have just created:
    
        `conda activate launcher`
    
    - When you are done with your session, you may deactivate your `conda` environment using the `deactivate` command:
    
        `conda deactivate`
   

2. Install the TAO Launcher Python package called `nvidia-tao` into the conda launcher environment:
    
    `conda activate launcher`
    
    `pip3 install nvidia-tao`

3. Invoke the entrypoints using the this command `tao -h`. You should see the following output:
```
usage: tao 
         {list,stop,info,augment,bpnet,classification,detectnet_v2,dssd,emotionnet,faster_rcnn,fpenet,gazenet,gesturenet,
         heartratenet,intent_slot_classification,lprnet,mask_rcnn,punctuation_and_capitalization,question_answering,
         retinanet,speech_to_text,ssd,text_classification,converter,token_classification,unet,yolo_v3,yolo_v4,yolo_v4_tiny}
         ...

Launcher for TAO

optional arguments:
-h, --help            show this help message and exit

tasks:
      {list,stop,info,augment,bpnet,classification,detectnet_v2,dssd,emotionnet,faster_rcnn,fpenet,gazenet,gesturenet,heartratenet
      ,intent_slot_classification,lprnet,mask_rcnn,punctuation_and_capitalization,question_answering,retinanet,speech_to_text,
      ssd,text_classification,converter,token_classification,unet,yolo_v3,yolo_v4,yolo_v4_tiny}
```

   For more info, visit the [TAO Toolkit documentation](https://docs.nvidia.com/tao/tao-toolkit/text/tao_toolkit_quick_start_guide.html).

4. Install other dependencies needed to run the lab:
```
pip install jupyterlab \
    matplotlib \
    fiftyone \
    attrdict \
    tqdm \
    gdown \
    nvidia-pyindex \
    tritonclient[all]
```

** Run the Labs**

Activate the conda launcher environment: `conda activate launcher`
    
You are to run the first two notebooks `1.Data_labeling_and_preprocessing.ipynb` and `2.Object_detection_using_TAO_YOLOv4.ipynb` in the `launcher` environment.

Launch the jupyter lab with:

`jupyter-lab --no-browser --allow-root --ip=0.0.0.0 --port=8888 --NotebookApp.token="" --notebook-dir=~/End-to-End-Computer-Vision/workspace` 

Remember to set the `--notebook-dir` to the location where the `project folder` where this material is located.

Then, open jupyter lab in the browser at http://localhost:8888 and start working on the lab by clicking on the `Start_here.ipynb` notebook.

When you are done with `1.Data_labeling_and_preprocessing.ipynb` and `2.Object_detection_using_TAO_YOLOv4.ipynb`, move to the next section.

##### Lab 3 

To start the Triton Inference Server instance, you will need to run a container along with the `launcher` virtual environment. This is to emulate the client-server mechanism but on the same system. To start the server, `open a new terminal` and launch the command:
```
docker run \
  --gpus=1 --rm \
  -p 8000:8000 -p 8001:8001 -p 8002:8002 \
  -v ~/End-to-End-Computer-Vision/workspace/models:/models \
  nvcr.io/nvidia/tritonserver:22.05-py3 \
  tritonserver \
  --model-repository=/models \
  --exit-on-error=false \
  --model-control-mode=poll \
  --repository-poll-secs 30
```
In order to work properly in this lab, the triton server version should match the TAO Toolkit version that was installed (visible by running `tao info`). Containers with the same `yy.mm` tag avoid version mismatches and conflicts that may prevent you from running and deploying your models. The path to the local model repository needs to be set as well in order to be mapped inside the container.

After starting Triton Server, you will see an output on the terminal showing `the server starting up and loading models`. This implies Triton is ready to accept inference requests.
```
+----------------------+---------+--------+
| Model                | Version | Status |
+----------------------+---------+--------+
| <model_name>         | <v>     | READY  |
| ..                   | .       | ..     |
| ..                   | .       | ..     |
+----------------------+---------+--------+
...
...
...
I1002 21:58:57.891440 62 grpc_server.cc:3914] Started GRPCInferenceService at 0.0.0.0:8001
I1002 21:58:57.893177 62 http_server.cc:2717] Started HTTPService at 0.0.0.0:8000
I1002 21:58:57.935518 62 http_server.cc:2736] Started Metrics Service at 0.0.0.0:8002
```

Now you can go back to your browser with jupyter lab open and run `3.Model_deployment_with_Triton_Inference_Server.ipynb`.

When you are done with the notebook, shut down jupyter lab by selecting `File > Shut Down` as well as the Triton Docker container of the server by pressing `ctrl + c` in the logs terminal. 


##### Lab 4 & 5

To run the DeepStream content, build a Docker container by following these steps:  

- Open a terminal window, navigate to the directory where `Dockerfile_deepstream` is located (e.g. `cd ~/End-to-End-Computer-Vision`)
- Run `sudo docker build -f Dockerfile_deepstream --network=host -t <imagename>:<tagnumber> .`, for instance: `sudo docker build -f Dockerfile_deepstream --network=host -t deepstream:1.0 .`
- Next, execute the command: `sudo docker run --rm -it --gpus=all -v ~/End-to-End-Computer-Vision/workspace:/opt/nvidia/deepstream/deepstream-6.1/workspace --network=host -p 8888:8888 deepstream:1.0`

flags:
- `--rm` will delete the container when finished.
- `-it` means run in interactive mode.
- `--gpus` option makes GPUs accessible inside the container.
- `-v` is used to mount host directories in the container filesystem.
- `--network=host` will share the host’s network stack to the container.
- `-p` flag explicitly maps a single port or range of ports.

When you are inside the container, launch jupyter lab: 
`jupyter-lab --no-browser --allow-root --ip=0.0.0.0 --port=8888 --NotebookApp.token="" --notebook-dir=/opt/nvidia/deepstream/deepstream-6.1/workspace`. 

Open the browser at `http://localhost:8888` and start working on `4.Model_deployment_with_DeepStream.ipynb` notebook. Then, move to `5.Measure_object_size_using_OpenCV.ipynb` and complete the material.

As soon as you are done with that, shut down jupyter lab by selecting `File > Shut Down` and the container by typing `exit` or pressing `ctrl d` in the terminal window.

Congratulations, you've successfully built and deployed an end-to-end computer vision pipeline!


#### Running Singularity Container

###### Lab 1 & 2

To build the TAO Toolkit Singularity container, run: `singularity build --fakeroot --sandbox tao_e2ecv.simg Singularity_tao`

Run the container with: `singularity run --fakeroot --nv -B ~/End-to-End-Computer-Vision/workspace:/workspace/tao-experiments tao_e2ecv.simg jupyter-lab --no-browser --allow-root --ip=0.0.0.0 --port=8888 --NotebookApp.token="" --notebook-dir=/workspace/tao-experiments`

The `-B` flag mounts local directories in the container filesystem and ensures changes are stored locally in the project folder. Open jupyter lab in browser: http://localhost:8888 

You may now start working on the lab by clicking on the `Start_here.ipynb` notebook.

When you are done with `1.Data_labeling_and_preprocessing.ipynb` and `2.Object_detection_using_TAO_YOLOv4.ipynb`, shut down jupyter lab by selecting `File > Shut Down` in the top left corner, then shut down the Singularity container by typing `exit` or pressing `ctrl + d` in the terminal window.


###### Lab 3

To download the Triton Inference Server Singularity container for the Server run: `singularity pull tritonserver:22.05-py3.sif docker://nvcr.io/nvidia/tritonserver:22.05-py3`

To build the Triton Inference Server Singularity container for the Client, run: `singularity build --fakeroot --sandbox triton_client_e2ecv.simg Singularity_triton`

To activate the Triton Inference Server container, run:
```
singularity run \
  --nv \
  -B ~/End-to-End-Computer-Vision/workspace/models:/models \
  /mnt/shared/bootcamps/tritonserver:22.05-py3.sif \
  tritonserver \
  --model-repository=/models \
  --exit-on-error=false \
  --model-control-mode=poll \
  --repository-poll-secs 30 \
  --http-port 8000 \
  --grpc-port 8001 \
  --metrics-port 8002
```

You may now activate the Triton Client container with: `singularity run --fakeroot --nv -B ~/End-to-End-Computer-Vision/workspace:/workspace triton_client_e2ecv.simg jupyter-lab --no-browser --allow-root --ip=0.0.0.0 --port=8888 --NotebookApp.token="" --notebook-dir=/workspace`

Then, open jupyter lab in browser: http://localhost:8888 and continue the lab by running `3.Model_deployment_with_Triton_Inference_Server.ipynb`.

**Note**

In a cluster environment, the `Triton Inference Server` container should be launched on the computing node(eg. dgx05) why the `Triton Client` container should be run on the login node (cpu). Therefore, within the notebook, url variable should be modified as follows:


```
assume you are on dgx05 then, replace

url = "localhost:8000" with url = "dgx05:8000" 

url = "localhost:8001" with url = "dgx05:8001"
```

As soon as you are done with that, shut down jupyter lab by selecting `File > Shut Down` and the Client container by typing `exit` or pressing `ctrl + d` in the terminal window.


###### Lab 4 & 5

To build the DeepStream Singularity container, run: `sudo singularity build --sandbox deepstream_e2ecv.simg Singularity_deepstream`

Run the DeepStream container with: `singularity run --fakeroot --nv -B ~/End-to-End-Computer-Vision/workspace:/opt/nvidia/deepstream/deepstream-6.1/workspace /mnt/shared/bootcamps/deepstream_e2ecv.simg jupyter-lab --no-browser --allow-root --ip=0.0.0.0 --port=8888 --NotebookApp.token="" --notebook-dir=/opt/nvidia/deepstream/deepstream-6.1/workspace`

Open jupyter lab in browser: http://localhost:8888 and complete the material by running `4.Model_deployment_with_DeepStream.ipynb` and `5.Measure_object_size_using_OpenCV.ipynb`.

Congratulations, you've successfully built and deployed an end-to-end computer vision pipeline!



## Known issues

### TAO

a. When installing the TAO Toolkit Launcher to your host machine’s native python3 as opposed to the recommended route of using a virtual environment, you may get an error saying that `tao binary wasn’t found`. This is because the path to your `tao` binary installed by pip wasn’t added to the `PATH` environment variable in your local machine. In this case, please run the following command:

`export PATH=$PATH:~/.local/bin`

b. When training, you can see an error message stating:
```
Resource exhausted: OOM when allocating tensor...
ERROR: Ran out of GPU memory, please lower the batch size, use a smaller input resolution, use a smaller backbone, or enable model parallelism for supported TLT architectures (see TLT documentation).
```
As the error says, you ran out of GPU memory. Try playing with batch size to reduce the memory footprint.

### NGC

You can see an error message stating:

`ngc: command not found ...`

You can resolve this by setting the path to ngc within the conda launcher environment as:

`echo "export PATH=\"\$PATH:$(pwd)/ngc-cli\"" >> ~/.bash_profile && source ~/.bash_profile`

### Triton Inference Server

You can see in the server logs an error message stating something similar to:

```
E0930 06:24:12.416803 1 logging.cc:43] 1: [stdArchiveReader.cpp::StdArchiveReader::40] Error Code 1: Serialization (Serialization assertion stdVersionRead == serializationVersion failed.Version tag does not match. Note: Current Version: 213, Serialized Engine Version: 205)
E0930 06:24:12.423693 1 logging.cc:43] 4: [runtime.cpp::deserializeCudaEngine::50] Error Code 4: Internal Error (Engine deserialization failed.)
```

The Server container is using a different version of TensorRT than the one the engine was generated with, so the Server is unable to load the model. Make sure to use containers with the same `<yy.mm>` tag when pulling from NGC as this ensures there are no version mismatches. You can verify the version of TAO by running the `tao info` command and then pull the appropriate `nvcr.io/nvidia/tritonserver:yy.mm-py3` Server container to solve the issue.

### DeepStream

You can see when running the pipeline an error similar to:

```
ERROR: [TRT]: 4: [runtime.cpp::deserializeCudaEngine::50] Error Code 4: Internal Error (Engine deserialization failed.)
ERROR: ../nvdsinfer/nvdsinfer_model_builder.cpp:1528 Deserialize engine failed from file: /opt/nvidia/deepstream/deepstream-6.1/workspace/yolo_v4/export/trt.engine
```
The DeepStream container uses a different version of TensorRT than the one the engine was generated with, so it is unable to use the TensorRT engine for inference. Please set the `tlt-encoded-model` path in the configuration file so that if the engine deserialization fails, DeepStream will attempt to rebuild the engine internally.
