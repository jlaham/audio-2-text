# Overview
Audio-2-Text is a serverless speech-to-text transcription pipeline built on Google Cloud Platform. The main element included in this pipeline is the Google Cloud Function code that is triggered by a Google Cloud Storage object change notification trigger.

## Components
The pipeline is built on three components:
1. Google Cloud Storage
1. Google Cloud Functions
1. Google Cloud Speech API

This pipeline also takes advantage of two more GCP services in order to automate the build process and abstract configuration variables by setting them in environment variables:
1. Google Cloud Container Builder
1. Google Cloud Runtime Configurator

## Build Process
For a first time build you would need to create a new runtime-config resource which is referenced below as ${CONFIG_NAME}.

`gcloud beta runtime-config configs create ${CONFIG_NAME} --description ${DESCRIPTION}`

Once created you will need to set up a Container Builder trigger using the `build/cloudbuild.yaml` declaration file. This file consists of a multi-step build process.
1. Set the environment variable `AUDIO_BUCKET` to the name of the GCS bucket where audio files will be uploaded.
1. Set the environment variable `TEXT_BUCKET` to the name of the GCS bucket where the Cloud Function will upload text files.
1. Deploy the Cloud Function

These are the substitutions that you must set in your build trigger
1. `_AUDIO_BUCKET`: The bucket where audio files will be uploaded
1. `_TEXT_BUCKET`: The bucket where the Cloud Function will upload text files
1. `_CONFIG_NAME`: The name of the runtime-config resource created above
1. `_FUNC_NAME`: The name of the Cloud Function
1. `_STAGE_BUCKET`: The staging bucket for the Cloud Function

N.B. The Cloud Function's service account needs object creator permissions on `_TEXT_BUCKET`.

## Workflow
The workflow is as follows:
1. Upload an audio file you want transcribed to the monitored Cloud Storage bucket
1. The upload of an object triggers an object change notification, which triggers a Cloud Function
1. The Cloud Function reads the uploaded file and makes an API call to the Cloud Speech API, referencing the audio file in the Cloud Storage bucket
1. Once finished processing, the Cloud Speech API returns a text file to the Cloud Function
1. The Cloud Function writes the text file to another bucket that isn't being monitored

![architecture diagram](/images/architecture.png)

## Caveat
Currently, the Cloud Function is hard-coded to only accept audio files with the following specifications:
- Format: FLAC
- Frequency: 44,100 Hz
- Channels: Mono (this is a Cloud Speech API requirement)
- Language: English (US)

## Deployment
Once that's done, all you need to do is upload an audio file to the monitored bucket, and follow the logs in your GCP console to see the status of the transcription. Once the Cloud Speech API service completes transcribing the file you should find a file in the text bucket with the same name as your original audio file, with a 'txt' extension instead.

Enjoy!
