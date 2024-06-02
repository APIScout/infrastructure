# Infrastructure

## Setup

### USE Model
Before spinning up the docker containers, some preparataion must be done. First of all, you must dowload the embedding model from Tensorflow. The model is the Universal Sentence Encoder (USE) model created by Google [^1]. By running the following commands, a new directory containing the model will be saved in the `models` directory (it is normal for the script to hang for a bit, the model is ~1GB).

```sh
conda env create --file=./scripts/environment.yml
conda run -n api-scout python ./scripts/download-use.py
```

### .env
In the `compose` directory you will find a `.template.env` file. Duplicate this file (renaming it to `.env`) and change it appropriately. This env file is used by the docker-compose files (for example to set the passwords of the dbs admin user).

### Config
In the `config` directory you will find a `template.config.yml` file. Duplicate this file and change it appropriately. This configuration file is used by the backend of API Scout to set some important variables. N.B. if you are using API Scout in release mode (the `GIN_MODE` env variable in the backend docker-compose file), the duplicated file should be renamed to `release.config.yml`. If you are using the debug mode, rename it to `debug.config.yml`.

## Running the Containers
To spin up the container and bring up the whole infrastructure, you must run the command below.

```sh
docker compose -f ./compose/backend.yml -f ./compose/elastic.yml -f ./compose/mongo.yml -p api-scout-infra up -d
```

## Troubleshooting

### Backend Container Crashes on Startup
If the `backend` container crashes as soon as it goes up, then check if the `config` directory contains either a `debug.config.yml` or `release.config.yml` (depending on what you have set for the `GIN_MODE` env variable in the backend docker-compose file).

### Illegal Instruction in Models Container
If you are using an Apple Silicon Mac, then you might encounter an error in which the `models` container refuses to start with the error `Illegal instruction`. In this case, go to the Docker Application > Settings > General and uncheck "Use Rosetta for x86_64/amd64 emulation on Apple Silicon". Now click "Apply & Restart", the issue should be fixed now.

### Cannot Access Model from Models Container
If you are not able to access the embedding model from the `models` container, make sure that you have [downloaded the USE model](#USE-Model), and that the `models` directory contains the embedding model.

### Elastic Container Crashes with Code 137
If the `elastic` container crashed unexpectedly with a 137 code, it could be that Docker runs out of memory. This container by default requires 6GB of RAM, but if you don't have enough, you can comment the line below (contained in the `elastic.yml` file).

```yml
- "ES_JAVA_OPTS=-Xms6000m -Xmx6000m"
```

## References
[^1] D. Cer, Y. Yang, S.-y. Kong, N. Hua, N. Limtiaco, R. St. John, N. Constant, M. Guajardo-Cespedes, S. Yuan, C. Tar, Y.-H. Sung, B. Strope, and R. Kurzweil, “Universal Sentence Encoder,” Mar. 2018. Publication Title: arXiv e-prints ADS Bibcode: 2018arXiv180311175C.
