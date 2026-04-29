Deep Multilingual Normalization
===============================

This repository hosts the code for our [Medical concept normalization in French using multilingual terminologies and contextual embeddings](https://doi.org/10.1016/j.jbi.2021.103684) article. It was recently reimplemented using [edsnlp](https://github.com/aphp/edsnlp).

If this method is useful to you, please consider citing our article, and/or giving a star to this repository :

```bibtex
@article{wajsburt2021medical,
    title = {Medical concept normalization in French using multilingual terminologies and contextual embeddings},
    journal = {Journal of Biomedical Informatics},
    volume = {114},
    pages = {103684},
    year = {2021},
    issn = {1532-0464},
    doi = {https://doi.org/10.1016/j.jbi.2021.103684},
    url = {https://www.sciencedirect.com/science/article/pii/S1532046421000137},
    author = {Perceval Wajsbürt and Arnaud Sarfati and Xavier Tannier},
    keywords = {Natural language processing, Information extraction, Medical concept normalization, Multilingual representation},
}
```

## Install

We recommend you use [`poetry`](https://python-poetry.org/docs/#installing-with-pipx) to install the dependencies from the lock file.

```bash
# Clone the repo
git clone https://github.com/percevalw/mlg_norm.git
cd mlg_norm

# Install the dependencies with poetry (or use pip otherwise)
poetry install
# pip install -e .
```

## Loading the UMLS

You will need to download the UMLS version to run this method. For instance, to replicate our results on the Quaero corpus, you will need the 2014AB version. Here are the steps to load the UMLS:

1. Download and unzip the `2014ab-1-meta.nlm` file (it's really a zip with a different extension) under the *2014AB UMLS Full Release Files* section at [https://www.nlm.nih.gov/research/umls/licensedcontent/umlsarchives04.html#2014AB_full](https://www.nlm.nih.gov/research/umls/licensedcontent/umlsarchives04.html#2014AB_full)
2. Enter the `2014AB/META` folder and unzip MRCONSO and MRSTY

    ```
    gunzip MRCONSO.RRF.*.gz MRSTY.RRF.*.gz
    ```
3. Concatenate the multiple MRCONSO files:

    ```
    cat MRCONSO.RRF.aa MRCONSO.RRF.ab > MRCONSO.RRF
    ```
4. Move `MRCONSO.RRF`, `MRSTY.RRF` and `resources/sty_groups.tsv` to the `data/umls/2014AB` folder.

## Train and evaluate a model

Our method is composed of three steps:

- Terminology preparation, to materialize the filtered UMLS rows as parquet fragments
  that EDS-NLP can read in parallel:

    ```bash
    python -m mlg_norm.train prepare_terminology --config configs/config.yml
    ```

- Pre-training, to learn multilingual representations and produce similar representation for synonyms of a same concept:

    ```bash
    python -m mlg_norm.train pretrain --config configs/config.yml
    ```

- Short classifier training. This will probe the pre-trained embedding and finetune the concepts weights.

    ```bash
    python -m mlg_norm.train train_classifier --config configs/config.yml
    ```

Finally, you can evaluate the model:

```bash
python -m mlg_norm.evaluate evaluate --config configs/config.yml
```

Consider changing the [`configs/config.yml`](/configs/config.yml) to fit your needs.
