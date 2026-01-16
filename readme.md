# Animal detection and counting in Aerial Imagery

This repository orchestrates four interconnected sub-repositories to enable Eider duck detection and analysis in aerial imagery.

## Sub-repositories

1. **OpenWildlife-ml-model**
   - Contains code based on `mmdetection`, tailored for animal detection in aerial imagery.

2. **OpenWildlife-ls-ML-backend**
   - Machine learning backend for Label Studio, supporting custom model inference and active learning.
   - Commit SHA: `e3ba93494120929c43670b842c320b015faf5a52` (referenced due to lack of official versioning).
   - [Repository link](https://github.com/HumanSignal/label-studio-ml-backend)

3. **OpenWildlife-frontend**
   - User interface for visualizing and interacting with detection results.

4. **OpenWildlife-ls-sdk**
   - SDK required for both frontend and backend builds.

---

## Installation

### Docker-based Deployment (Recommended)

Docker simplifies setup and ensures consistency. [Install Docker](https://docs.docker.com/engine/install/) if you haven’t already.

Make sure Docker engine is up and running for Linux and Docker Desktop for Windows.

#### 1. Clone the Repository with Submodules

Clone the main repository along with all submodules:

```bash
git clone --recurse-submodules https://github.com/echonax07/OpenWildlife.git
```

If you've already cloned without submodules, initialize them with:

```bash
git submodule update --init --recursive
```

**Note for Windows users:** Clone with symlink support to avoid issues mentioned [here](https://github.com/HumanSignal/label-studio/issues/4971):

```bash
git clone -c core.symlinks=true --recurse-submodules https://github.com/echonax07/OpenWildlife.git
```

This will clone all four submodules:
- `OpenWildlife-frontend`
- `OpenWildlife-ls-ML-backend`
- `OpenWildlife-ls-sdk`
- `OpenWildlife-ml-model`

#### 2. Create a Shared Docker Network

For Linux/Windows
```bash
docker network create labelstudio_shared
```

This network allows frontend and backend containers to communicate.

#### 3. Build and Run the Frontend


```bash
# For Linux/Windows
cd OpenWildlife-frontend
docker build -t label-studio:latest .
docker compose up
```

#### 4. Build and Run the ML Backend

For Linux/Windows
```bash
cd ../OpenWildlife-ml-model
```

Switch the branch to grounding_dino
```bash
git checkout grounding_dino
```

Build the image

```bash
# For Linux/Windows
docker build -t OpenWildlife-ml-model:latest .
```

Copy the downloaded checkpoints folder (inside the docker image) to `./checkpoints/` folder on local filesystem.

```bash
# For Linux
sudo docker run --rm   --user $(id -u):$(id -g)   -v $(pwd):/host   OpenWildlife-ml-model:latest   cp -r /app/checkpoints /host/
```

```bash
:: For windows Powershell
docker run --rm -v "${PWD}:/host" OpenWildlife-ml-model:latest cp -r /app/checkpoints /host/
```

Before proceeding to next step, ensure that :
- The frontend is running.

- Copy the API key/Access token from the Frontend webpage (http://localhost:8080) under `Account Settings`.

- Open docker-compose.yml in `OpenWildlife-ml-model` folder and replace the following placeholders with actual values:
  - `LABEL_STUDIO_API_KEY`: Replace with the API key copied from the frontend.

Start the ML backend:

For Linux/Windows
  ```bash
  docker compose up
  ```

Both frontend and backend should now be running. Verify by accessing the frontend in your browser.

---

#### 5.Setting up a Project in Label Studio
- Open Label Studio Frontend in your browser (http://localhost:8080).
- Create a new project.
- Open the project, go to settings -> labelling interface -> Click on code and paste the following XML. (Modify the classes )
```xml
<View>
 
<Image name="image" value="$image" zoom="true" zoomControl="true" rotateControl="false" contrastControl="true" brightnessControl="true" maxWidth="1500px"/>

<!--Modify the classes according to your dataset-->
<KeyPointLabels name="keypoints" toName="image">  
<Label value="female duck" background="#EF4444"/>
<Label value="male duck" background="blue"/>
<Label value="Ice" background="green"/>
<Label value="Juvenile duck" background="yellow"/>
<Label value="duck" background="orange"/>
</KeyPointLabels>


<!-- DO NOT CHANGE ANYTHING BELOW THIS -->
<RectangleLabels name="rectangle" toName="image">  
<Label value="rectangle" background="black"/>
</RectangleLabels>  

<PolygonLabels name="label" toName="image" strokeWidth="3" pointSize="small" opacity="0.36" snap="pixel1">
    <Label value="Train region" background="#c800ff"/>
 </PolygonLabels>

  <!-- Radio button to choose whether to train on the whole image or train region -->
  <Choices name="train_region_choice" toName="image" choice="single-radio" showInline="true" required="true">
    <Choice value="Train only on train region"/>
    <Choice value="Train on whole image"/>
  
  </Choices>

  <!-- Add numeric input fields -->
  
   <TextArea name="width" toName="image" placeholder="Enter width (float)" inputType="number" required="true" perRegion="false"/>
  
  <TextArea name="height" toName="image" placeholder="Enter height (float)" inputType="number" required="true" perRegion="false"/>
            
</View>

```

- GO to settings -> Machine Learning -> Connect ML model, put the name of your desire and the url as `http://mmdetection:9090`

Now everything should be set up. You can start annotating images and use the ML backend for predictions and training.


## Development Setup

### 1. Setting up `OpenWildlife-ml-model`

We recommend using Python’s `virtualenv` (or `Miniconda` if preferred):

```bash
cd OpenWildlife-ml-model
bash compute_canada/submit/create_env_from_requirements.sh <env_name>
source ~/<env_name>/bin/activate
```

### 2. Installing `OpenWildlife-frontend`

With your environment activated:

```bash
cd ../OpenWildlife-frontend
pip install -v -e .
```

### 3. Installing `label-studio-ml-backend`

With the same environment activated:

```bash
cd ../label-studio-ml-backend
pip install -v -e .
```

---

## Notes

- Replace `<env_name>` with your desired environment name.
- `~/` refers to your home directory.
- Ensure all dependencies are installed as per each repository’s requirements.

---
