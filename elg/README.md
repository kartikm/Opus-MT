# Creating European Language Grid -compatible Docker images

## Prerequisites

You have to be registered as a provider with ELG (meaning, have an account in the ELG portal with that status), and you have to have a Dockerhub account and be a member of the helsinkinlp team there.

## The server

The file `elg/elg_server.py` imports code from the main `server.py` and adds some ELG compatibility on top.

## The automatic part

Almost everything is done by the bash script `build_elg_image_and_metadata.sh`. It is configured as follows:

1. Model selection
   * If the directory `models/` exists, it is used as-is. It should contain subdirectories like `deu-eng/`, with `README.md` included. The script will parse the READMEs and discover all the language pairs the models provide. 
   * If it doesn't, you can write a file `models.txt` with one URL per line, indicating download locations for Opus-MT or Tatoeba models. These are automatically downloaded and unzipped, and we proceed as above.
2. Docker image name. You can supply this as an argument, eg. `opus-mt-elg-deu-eng`, but there's a reasonable default (alphabetically sort all language ids and construct a name from them).

The script will:

1. Download the models if they don't already exist and name their directories according to language pairs
2. Construct the translation pairs
3. Generate as many XML files as there are language pairs, using `generate_metadata.py`
4. Write configuration files for the server `write_configuration.py`, which knows about multilingual models
5. Build and tag the docker image `opus-mt-base` and then the one for this ELG model using `elg/Dockerfile`, tagged as in 2. above
6. Ask if you want to login to Dockerhub now, and if so, push the image
7. Remind you to upload the XML to ELG

### Example

So far, our ELG images have all had both directions of one pair, eg. fin-eng and eng-fin. As a different example, if you write into `models.txt` the following lines:

```
https://object.pouta.csc.fi/Tatoeba-MT-models/dan-deu/opus-2021-02-18.zip
https://object.pouta.csc.fi/Tatoeba-MT-models/dan-eng/opus-2021-01-03.zip
```

and run `build_elg_image_and_metadata.sh`, this will result in an image that provides dan-deu and dan-eng translation, each with its own endpoint and `marian-server`.

If you write

```
https://object.pouta.csc.fi/Tatoeba-MT-models/dan-deu/opus-2021-02-18.zip
https://object.pouta.csc.fi/Tatoeba-MT-models/bel+rus+ukr-bel+rus+ukr/opus-2020-06-16.zip
```

you will get dan-deu in one server that doesn't use input-initial language tags, and another server with bel-rus, bel-ukr, rus-bel, rus-ukr, ukr-bel and ukr-rus, each with their own endpoint and input-initial language tag in another, all running in the same container.

### Notes

The script will also count how many models are in the image, and write into the metadata an amount of memory that should be enough for running them simultaneously (768M per model).

If you want to bump the version number, do that by editing the script manually.

## The manual part

When the script has finished, there remains a manual task: publishing the image and metadata on ELG. The following procedure applies as of June 2021, but will probably change from time to time.

1. Point your web browser at the [ELG catalogue](https://live.european-language-grid.eu/catalogue/)
2. Log in by clicking on the small icon on the top-right (you have to be registered as a provider)
3. Click on "My grid" on the top panel
4. Hover on the "Add items" menu and select "Upload XML".
5. Select "Upload multiple items".
6. Check the box for ELG-compatible service, and choose `metadata_for_elg.zip` which the build script should have left in the `elg` directory.
7. Hopefully the upload completes successfully. If not, the metadata specification has probably changed again. It will take some time for the system to process these.
8. When processing has finished (you may get an email), click on "My items", find the entries you've just uploaded, select them, and choose the action "Publish".
