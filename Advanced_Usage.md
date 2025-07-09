## Advanced Usage

### Generation settings

Tortoise is primarily an autoregressive decoder model combined with a diffusion model. Both of these have a lot of knobs
that can be turned that I've abstracted away for the sake of ease of use. I did this by generating thousands of clips using
various permutations of the settings and using a metric for voice realism and intelligibility to measure their effects. I've
set the defaults to the best overall settings I was able to find. For specific use-cases, it might be effective to play with
these settings (and it's very likely that I missed something!)

These settings are not available in the normal scripts packaged with Tortoise. They are available, however, in the API. See
```api.tts``` for a full list.

### Prompt engineering

Some people have discovered that it is possible to do prompt engineering with Tortoise! For example, you can evoke emotion
by including things like "I am really sad," before your text. I've built an automated redaction system that you can use to
take advantage of this. It works by attempting to redact any text in the prompt surrounded by brackets. For example, the
prompt "\[I am really sad,\] Please feed me." will only speak the words "Please feed me" (with a sad tonality).

### Playing with the voice latent

Tortoise ingests reference clips by feeding them through individually through a small submodel that produces a point latent,
then taking the mean of all of the produced latents. The experimentation I have done has indicated that these point latents
are quite expressive, affecting everything from tone to speaking rate to speech abnormalities.

This lends itself to some neat tricks. For example, you can combine feed two different voices to tortoise and it will output
what it thinks the "average" of those two voices sounds like.

#### Generating conditioning latents from voices

Use the script `get_conditioning_latents.py` to extract conditioning latents for a voice you have installed. This script
will dump the latents to a .pth pickle file. The file will contain a single tuple, (autoregressive_latent, diffusion_latent).

Alternatively, use the api.TextToSpeech.get_conditioning_latents() to fetch the latents.

#### Using raw conditioning latents to generate speech

After you've played with them, you can use them to generate speech by creating a subdirectory in voices/ with a single
".pth" file containing the pickled conditioning latents as a tuple (autoregressive_latent, diffusion_latent).

## Tortoise-detect

Out of concerns that this model might be misused, I've built a classifier that tells the likelihood that an audio clip
came from Tortoise.

This classifier can be run on any computer, usage is as follows:

```commandline
python tortoise/is_this_from_tortoise.py --clip=<path_to_suspicious_audio_file>
```

This model has 100% accuracy on the contents of the results/ and voices/ folders in this repo. Still, treat this classifier
as a "strong signal". Classifiers can be fooled and it is likewise not impossible for this classifier to exhibit false
positives.

## Model architecture

Tortoise TTS is inspired by OpenAI's DALLE, applied to speech data and using a better decoder. It is made up of 5 separate
models that work together. I've assembled a write-up of the system architecture here:
[https://nonint.com/2022/04/25/tortoise-architectural-design-doc/](https://nonint.com/2022/04/25/tortoise-architectural-design-doc/)

## Training

These models were trained on my "homelab" server with 8 RTX 3090s over the course of several months. They were trained on a dataset consisting of
~50k hours of speech data, most of which was transcribed by [ocotillo](http://www.github.com/neonbjb/ocotillo). Training was done on my own
[DLAS](https://github.com/neonbjb/DL-Art-School) trainer.

I currently do not have plans to release the training configurations or methodology. See the next section..

## Looking forward

Tortoise v2 is about as good as I think I can do in the TTS world with the resources I have access to. A phenomenon that happens when
training very large models is that as parameter count increases, the communication bandwidth needed to support distributed training
of the model increases multiplicatively. On enterprise-grade hardware, this is not an issue: GPUs are attached together with
exceptionally wide buses that can accommodate this bandwidth. I cannot afford enterprise hardware, though, so I am stuck.

I want to mention here
that I think Tortoise could be a **lot** better. The three major components of Tortoise are either vanilla Transformer Encoder stacks
or Decoder stacks. Both of these types of models have a rich experimental history with scaling in the NLP realm. I see no reason
to believe that the same is not true of TTS.
