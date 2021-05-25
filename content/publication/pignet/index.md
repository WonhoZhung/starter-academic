---
title: "PIGNet: A physics-informed deep learning model toward generalized drug-target interaction predictions"

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here 
# and it will be replaced with their full name and linked to their profile.
authors:
- admin
- Seokhyun Moon
- Soojung Yang
- Jaechang Lim
- Wooyoun Kim

# Author notes (optional)
author_notes:
- "Equal contribution"
- "Equal contribution"
- "Equal contribution"

date: "2020-08-22T00:00:00Z"
doi: "arXiv:2008.12249"

# Schedule page publish date (NOT publication's date).
publishDate: "2020-08-22T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["3"]

# Publication name and optional abbreviated publication name.
publication: In *arXiv*
publication_short: In *arXiv*

abstract: Recently, deep neural network (DNN)-based drug-target interaction (DTI) models are highlighted for their high accuracy with affordable computational costs. Yet, the models' insufficient generalization remains a challenging problem in the practice of in-silico drug discovery. We propose two key strategies to enhance generalization in the DTI model. The first one is to integrate physical models into DNN models. Our model, PIGNet, predicts the atom-atom pairwise interactions via physics-informed equations parameterized with neural networks and provides the total binding affinity of a protein-ligand complex as their sum. We further improved the model generalization by augmenting a wider range of binding poses and ligands to training data. PIGNet achieved a significant improvement in docking success rate, screening enhancement factor, and screening success rate by up to 2.01, 10.78, 14.0 times, respectively, compared to the previous DNN models. The physics-informed model also enables the interpretation of predicted binding affinities by visualizing the energy contribution of ligand substructures, providing insights for ligand optimization. Finally, we devised the uncertainty estimator of our model's prediction to qualify the outcomes and reduce the false positive rates.

# Summary. An optional shortened abstract.
# summary: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis posuere tellus ac convallis placerat. Proin tincidunt magna sed ex sollicitudin condimentum.

tags: ["Drug-Target Interaction", "Graph Neural Network", "Physics-informed Model", "Deep Learning"]

# Display this page in the Featured widget?
featured: true

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

url_pdf: 'https://arxiv.org/pdf/2008.12249.pdf'
url_code: ''
url_dataset: ''
url_poster: ''
url_project: ''
url_slides: ''
url_source: ''
url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
image:
  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/pLCdAaMFLTE)'
  focal_point: ""
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects: []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: ""


