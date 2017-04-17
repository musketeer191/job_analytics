# Job and Skill Analytics

This repo contains source codes for analyzing data of job postings to gain insights on jobs and skills required for different jobs and sectors. The codes are written in `Jupyter` notebooks and all helper functions are centralized in `ja_helpers.py` module. In addition, there are some more helpers in `my_util.py`, `stat_helpers.py` and `parse_funs.py`.

## Main components
Main components handle the following steps in the pipeline: i) __Preprocess job posts__ and meta-data, ii) __Extract features__ for topic and matrix factorization (MF) models, iii) __Cluster skills__ using the models, iv) __Connect job titles__ based on their topic similarity and v) __Determine consistency__ among job posts of a given job title.

Following are details on each component.

1. __Preprocess__. 
This step is handled by `preprocess` and `parse_title` notebooks.

`parse_title` parses each job titles into separate components: position, domain and primary function (some job titles may also have secondary function). This parsing serves two goals: 
  + grouping job titles by their domains or functions (or by position).
  + standardizing job titles to unify different forms of the same job title e.g. Software Engineer and Engineer, Software will be unified as Software Engineer.
The script uses the parser from https://jobsense.sg/api/get/job-title-parser/.

`preprocess` handles the following tasks:
  + filter posts containing only 1 skill
  + filter skills occuring in only 1 post
  + filter stop-word skills, e.g. business
In addition, there is also one part for cleaning data on employers.

2. __Extract features__ required by topic and MF models.

Each job post is regarded as a document and each skill is an entry in the vocabulary of skills.
The features for LDA are represented by a document-skill matrix whose each entry _e(d, s)_ is occurrence count of skill _s_ in document _d_. As the skills can be uni-, bi- or tri-grams, there is one important difference from counting uni-gram words in documents: __counts of certain skills can be over-estimated__!!! For example, count of _programming_, as a skill itself, can be inflated as it also appears in other skills including 'Java programming', 'Python programming' and so on.

The script in `extract_feat.ipynb` handles this problem by first counting occurence of longer n-grams. Once the couting is done, it removes longer n-grams from documents and go on to count shorter n-grams. The removal gets rid of the over-estimation.

3. __Cluster skills__ using LDA.

I adopt the module for LDA in scikit-learn. The script is in `cluster_skill.ipynb`.

4. __Connect job titles__.

Given topic distribution of each job post learnt by LDA, we can compute topic similarity score between any two job posts as the similarity of their topic distributions. 

Given two job titles t and t' with their sets of job posts P(t) and P(t'), we define similarity between t and t' as the average of pairwise similarity scores of pairs of posts (p, p') where p and p' belong to P(t) and P(t') respectively.

Similarity scores between pairs of job titles under the same domain are precomputed by the script in `sim_by_domain.ipynb`. Similarly, similarity scores between pairs of job titles with same primary function are precomputed in `sim_by_func.ipynb`.

5. __Determine consistency among job posts.__

We define consistency score for a given set of job posts as the average of pairwise similarity scores over all pairs of posts from the set.

Given a job title, we then define its consistency score as the consistency score of the set of its posts. The consistency score allows us to profile a job title as _general_ or _niche_ job title (e.g. Manager is general while Software Engineer is niche). Computing consistency scores is done in `job_consistency.ipynb`.