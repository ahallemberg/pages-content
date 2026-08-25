In the summer of 2024 I worked as a machine learning engineer intern at [Q-Free](https://www.q-free.com/) in Trondheim, synthesizing weather conditions into images from ALPR cameras, the cameras that read license plates on toll roads. The work continued part time through the autumn, alongside my studies, until December.

## The problem

Q-Free builds tolling and intelligent transport systems, and its cameras have to read license plates in whatever the sky delivers: rain, fog, snow, low sun. The models doing the reading are trained on images, and most of the images you can collect are taken in fine weather. You cannot order a week of fog to balance a dataset, so bad weather is exactly the data such models see least of and need most.

My internship grew out of a topic the company had proposed as a master's thesis: investigate whether AI can synthesize weather into existing camera images realistically enough that the synthetic images work as training data. I took it on as a summer intern instead.

## Two ways to make weather

The learned approach was CycleGAN-Turbo, a one-step image-to-image translation model that I fine-tuned to translate images between clear and degraded weather. An unpaired translation model is the natural fit here, because the data is unpaired by nature: there are plenty of clear images and some foggy ones, but never the same scene captured in both, so the model has to learn the mapping without matched pairs.

The second approach did not learn the weather at all. Fog thickens with distance, so if you know how far away every pixel is, you can composite fog that behaves correctly. I used a monocular depth estimation model, Depth Anything, to predict a depth map for each image, then layered fog over the image with OpenCV, scaled by depth. It is a much simpler method, and that is what made it useful: a trained generator has to beat this baseline to be worth its training cost.

## Training in the cloud

Training ran on Google Cloud. I built a small harness around Vertex AI custom training jobs: each experiment is a folder with an entrypoint, described in one config file, and three scripts handle logging in, building and pushing the Docker image to Artifact Registry, and submitting the training job to a GPU instance. Adding a new experiment meant adding an entry to the config, not writing new infrastructure.

## An inference platform on Kubernetes

Once there were trained models, the rest of the summer went into running them at scale. I set up a Google Kubernetes Engine cluster with separate CPU and GPU node pools, both autoscaling from zero, so the cluster costs nothing while idle. Jobs were orchestrated with Argo Workflows: point a workflow at an input bucket and an output bucket in Cloud Storage, and the cluster spins up nodes, runs every image through a model, writes the results and scales back down.

Inside each pod the work is a pipeline built on Python multiprocessing: preprocessing workers on CPU feed a queue, a single GPU process consumes it, and postprocessing workers write the results out. The queues exist to keep the GPU busy, because GPU time is the expensive part and it should never sit idle waiting for image decoding.

To make the platform usable by more people than me, I wrapped it in a command line tool: creating the cluster, building and pushing the images, installing Argo and submitting workflows are single commands that check their own prerequisites, instead of pages of gcloud and kubectl incantations.

## The autumn

By the end of the summer the synthetic weather was convincing to the eye, and promising enough to take further. I stayed on part time through December, and the focus shifted from the models to the workflow around them: building a CI/CD pipeline from training through to inference, so that an updated model could be deployed without manual steps in between. When I left, the system was on its way into production. The real test, training a plate reader on the synthetic images and measuring the difference, was still ahead.

## What it taught me

Q-Free was my first machine learning job in industry, and I got it after my first year at NTNU. The machine learning background I brought was the perception group of [Ascend NTNU](https://pages.askhb.no/Ascend), the student organization that builds autonomous drones, twenty hours a week of it, and that was what got me in the door. What industry added is the lasting lesson: how much of machine learning is not the model. Designing and fine-tuning networks was a small share of the hours. Most of the work was everything around them: containers, GPU scheduling, storage, autoscaling, and the automation that lets one person operate all of it. I came in knowing PyTorch, and I left understanding why machine learning engineering is mostly engineering.
