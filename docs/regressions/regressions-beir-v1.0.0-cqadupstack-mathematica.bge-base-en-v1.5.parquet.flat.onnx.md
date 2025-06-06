# Anserini Regressions: BEIR (v1.0.0) &mdash; CQADupStack-mathematica

**Model**: [BGE-base-en-v1.5](https://huggingface.co/BAAI/bge-base-en-v1.5) with flat indexes (using ONNX for on-the-fly query encoding)

This page describes regression experiments, integrated into Anserini's regression testing framework, using the [BGE-base-en-v1.5](https://huggingface.co/BAAI/bge-base-en-v1.5) model on [BEIR (v1.0.0) &mdash; CQADupStack-mathematica](http://beir.ai/), as described in the following paper:

> Shitao Xiao, Zheng Liu, Peitian Zhang, and Niklas Muennighoff. [C-Pack: Packaged Resources To Advance General Chinese Embedding.](https://arxiv.org/abs/2309.07597) _arXiv:2309.07597_, 2023.

In these experiments, we are using ONNX to perform query encoding on the fly.

The exact configurations for these regressions are stored in [this YAML file](../../src/main/resources/regression/beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5.parquet.flat.onnx.yaml).
Note that this page is automatically generated from [this template](../../src/main/resources/docgen/templates/beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5.parquet.flat.onnx.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead and then run `bin/build.sh` to rebuild the documentation.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```
python src/main/python/run_regression.py --index --verify --search --regression beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5.parquet.flat.onnx
```

All the BEIR corpora, encoded by the BGE-base-en-v1.5 model and stored in Parquet format, are available for download:

```bash
wget https://rgw.cs.uwaterloo.ca/pyserini/data/beir-v1.0.0-bge-base-en-v1.5.parquet.tar -P collections/
tar xvf collections/beir-v1.0.0-bge-base-en-v1.5.parquet.tar -C collections/
```

The tarball is 127 GB and has MD5 checksum `5f8dce18660cc8ac0318500bea5993ac`.
After downloading and unpacking the corpora, the `run_regression.py` command above should work without any issue.

## Indexing

Sample indexing command, building flat indexes:

```
bin/run.sh io.anserini.index.IndexFlatDenseVectors \
  -threads 16 \
  -collection ParquetDenseVectorCollection \
  -input /path/to/beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5 \
  -generator DenseVectorDocumentGenerator \
  -index indexes/lucene-flat.beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5/ \
  >& logs/log.beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5 &
```

The path `/path/to/beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5/` should point to the corpus downloaded above.

## Retrieval

Topics and qrels are stored [here](https://github.com/castorini/anserini-tools/tree/master/topics-and-qrels), which is linked to the Anserini repo as a submodule.

After indexing has completed, you should be able to perform retrieval as follows:

```
bin/run.sh io.anserini.search.SearchFlatDenseVectors \
  -index indexes/lucene-flat.beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5/ \
  -topics tools/topics-and-qrels/topics.beir-v1.0.0-cqadupstack-mathematica.test.tsv.gz \
  -topicReader TsvString \
  -output runs/run.beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5.bge-flat-onnx.topics.beir-v1.0.0-cqadupstack-mathematica.test.txt \
  -encoder BgeBaseEn15 -hits 1000 -removeQuery -threads 16 &
```

Evaluation can be performed using `trec_eval`:

```
bin/trec_eval -c -m ndcg_cut.10 tools/topics-and-qrels/qrels.beir-v1.0.0-cqadupstack-mathematica.test.txt runs/run.beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5.bge-flat-onnx.topics.beir-v1.0.0-cqadupstack-mathematica.test.txt
bin/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.beir-v1.0.0-cqadupstack-mathematica.test.txt runs/run.beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5.bge-flat-onnx.topics.beir-v1.0.0-cqadupstack-mathematica.test.txt
bin/trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.beir-v1.0.0-cqadupstack-mathematica.test.txt runs/run.beir-v1.0.0-cqadupstack-mathematica.bge-base-en-v1.5.bge-flat-onnx.topics.beir-v1.0.0-cqadupstack-mathematica.test.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **nDCG@10**                                                                                                  | **BGE-base-en-v1.5**|
|:-------------------------------------------------------------------------------------------------------------|-----------|
| BEIR (v1.0.0): CQADupStack-mathematica                                                                       | 0.3163    |
| **R@100**                                                                                                    | **BGE-base-en-v1.5**|
| BEIR (v1.0.0): CQADupStack-mathematica                                                                       | 0.6922    |
| **R@1000**                                                                                                   | **BGE-base-en-v1.5**|
| BEIR (v1.0.0): CQADupStack-mathematica                                                                       | 0.8810    |

The above figures are from running brute-force search with cached queries on non-quantized flat indexes.
With ONNX query encoding on non-quantized flat indexes, observed results may differ slightly (typically, lower), but scores should generally be within 0.001 of the results reported above (with some outliers).
