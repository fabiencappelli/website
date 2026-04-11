---
title: Big Data Cloud Project — PySpark Pipeline on AWS for Image Recognition (Fruits-360) (OpenClassrooms Project 11)
slug: OCSpark
summary: Design and deployment of a scalable Big Data pipeline for large-scale image processing, automated visual feature extraction, and dimensionality reduction, following a production-oriented data engineering and machine learning workflow.
stack: [Python, AWS S3, EMR, Spark, PySpark, JupyterHub, MobileNetV2, PCA]
featured: true
order: 4
---

## Context

The initial idea was a biodiversity awareness application based on **fruit recognition**.  
The challenge is typical of a real-world scenario: a pipeline that works on a local machine eventually stops scaling when data volume increases.

This project therefore answers a simple question:

> How do you move from a classic “data science” notebook to a distributed, scalable, and cost-controlled cloud execution?

---

## Dataset

I used the **Fruits-360** dataset (Mihai Oltean, 2017), specifically the test set.

- **22,819 images**
- **131 classes** (fruit varieties: apples, pears, etc.)
- RGB images, **100×100** resolution
- Labels derived from the folder structure (directory names)

This dataset is a strong benchmark because it allows pipeline industrialization without relying on an external API or manual annotations.

---

## Target Architecture

The pipeline relies on a simple but robust cloud architecture:

- **S3**: Data Lake (raw images + outputs)
- **EMR**: Spark/Hadoop cluster
- **IAM**: access management (and compliance constraints)
- **JupyterHub**: PySpark notebook interface on the Master Node

The overall workflow is:

<div class="mermaid">
flowchart TD
  A["S3 (images)"] --> B
  B["EMR Spark (feature extraction + PCA)"] --> C["S3 (reduced features)"]
</div>

---

## AWS Environment Setup

The implementation included all the standard deployment steps:

1. Creation of an S3 bucket and image upload
2. IAM configuration (roles + policies, least-privilege approach)
3. Launch of an EMR cluster: **1 Master + 2 Workers**
4. Setup of a proxy tunnel to access JupyterHub
5. JupyterHub connection
6. Execution of the PySpark notebook
7. Saving outputs back to S3

---

## Distributed Processing with PySpark: Full Pipeline

### 1) Reading Images in Spark

Images are loaded using Spark’s `binaryFile` format, allowing the dataset to be handled as a distributed DataFrame.

- Recursive file loading
- Label column creation from the file path (`path`)
- Partitioning for parallel execution

---

### 2) Feature Extraction with MobileNetV2 (ImageNet Weights)

The core step is transforming each image into a feature vector (embedding).

I used:

- **Pre-trained MobileNetV2 on ImageNet**
- Removal of the classification head (keeping the final convolutional output)
- Inference mode only (frozen weights)

The key point here is that the model is not retrained: it is used as a **feature extractor**, which is common in production when the goal is to:

- standardize representations
- reduce costs
- avoid heavy retraining cycles

---

### 3) Weight Broadcasting (Spark Optimization)

In a Spark cluster, loading a deep learning model independently on every worker can dramatically increase memory usage and execution time.

To avoid this, I implemented a standard Spark strategy:

- **broadcasting model weights**
- rebuilding the model on each worker using the shared weights

This is an important point because it demonstrates real distributed execution understanding: not just “running a notebook on EMR,” but actively optimizing the workload distribution.

---

### 4) Dimensionality Reduction (StandardScaler + PCA)

The extracted embeddings are then standardized and reduced using PCA.

Spark ML pipeline:

- `StandardScaler`
- `PCA`

Goals:

- reduce data size
- improve visualization
- prepare future downstream models (classification, clustering, similarity search)

---

### 5) Saving Results to S3

The final output (reduced features + labels + path) is saved to S3 in **Parquet** format, a standard for Big Data environments.

---

## Cost Control (Realistic Cloud Approach)

I intentionally included cost optimization as part of the project, as would be expected in a business environment.

- Region: **eu-west-1 (Ireland)**
- Automatic shutdown after testing
- Total cost: **< €10**

📌 This matters because many cloud projects fail in practice not for technical reasons, but because costs spiral out of control.

---

## Results

This project produces a directly usable output:

- **22,819 processed images**
- **131 classes**
- Embeddings extracted with MobileNetV2
- Final PCA-reduced dataset
- Results stored on S3 in Parquet format

---

## Conclusion

This project allowed me to work through a complete **Cloud + Big Data + ML** pipeline, from ingestion and storage to distributed transformation and data preparation for future models.

It reflects the kind of workflow commonly found in real-world use cases: computer vision, product catalogs, quality control, or image similarity search, all with a clear need for **scalability**.
