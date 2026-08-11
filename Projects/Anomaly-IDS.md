Repo: https://github.com/YapiSamuel/Anomaly-IDS-RandomForest

# 🤖 Anomaly-Based Intrusion Detection — Random Forest on UNSW-NB15

**Complete. Trained and evaluated end to end on a public benchmark dataset.**

This project answers a question every detection team eventually has to settle: *can a model learn the shape of normal network traffic well enough to flag what isn't?* A Random Forest classifier is trained on the UNSW-NB15 dataset to separate benign traffic from attacks, following the full pipeline — preprocessing, feature selection, training, evaluation on held-out data, confusion matrix analysis, and feature-importance visualisation. Rather than stopping at an accuracy number, the work looks at *which* features carry the signal and *where* the model gets things wrong.

## 🔍 Features
- Classifies network traffic as normal or attack using a Random Forest of 100 estimators, trained on the UNSW-NB15 benchmark from the Australian Centre for Cyber Security — tens of thousands of labelled network flows described by connection, timing and protocol features
- Complete preprocessing pipeline in pandas: loads the training and testing sets, drops categorical and identifier fields (`id`, `attack_cat`, `proto`, `service`, `state`), removes null records, and splits features from labels
- Evaluated on a genuinely held-out testing set rather than a split of the training data, so the reported figure reflects unseen traffic
- Confusion matrix analysis showing not just how often the model is right but *how* it is wrong — high recall on attacks, weaker precision on normal traffic
- Feature-importance ranking that surfaces the connection-tracking fields carrying most of the signal: `ct_dst_src_ltm`, `std`, `ct_dst_sport_ltm`, `sbytes` and `ct_state_ttl`
- Exports flagged records to `detected_anomalies.csv`, and separates the pipeline into distinct scripts for preprocessing, training and visualisation

## 🧰 Skills Demonstrated
- **Reading a model honestly:** the classifier reaches roughly 98% on training data and roughly 90% on the held-out test set. The gap between those two numbers is the interesting part — it is the difference between what the model memorised and what it actually learned, and quoting only the first would misrepresent the work
- **Understanding what a false positive costs:** recall on attacks is strong, but precision on normal traffic is weaker, which in a real SOC means benign flows landing in an analyst's queue. In production that ratio, not raw accuracy, decides whether a detection is deployable or ignored
- **Feature interpretation as a security question:** the highest-ranked features are connection-tracking counters rather than raw byte volumes, which says something real about how this dataset's attacks distinguish themselves
- Supervised machine learning with scikit-learn: Random Forest classification, train/test methodology, and reproducible results through a fixed random seed
- Data preparation on a realistic security dataset — handling mixed-type columns, dropping fields that would leak the label, and cleaning missing records
- Visualisation with seaborn and matplotlib to make model behaviour legible rather than leaving it as a score
- Python 3, pandas, scikit-learn, working in a Linux environment with Vim

## 🎯 Outcome
A working intrusion detection model with results I can defend rather than just quote. The most useful lesson was that accuracy is the least interesting number in the report: a detection that catches most attacks but floods the queue with false positives will be turned off by the analysts it was built for, so the confusion matrix matters more than the headline figure. Working through the pipeline also made concrete why feature selection is a security decision and not just a modelling one — dropping `attack_cat` matters because leaving it in would let the model cheat. The project stands as a complete piece of work and a foundation for the obvious next steps: hyperparameter tuning, real-time scoring on live traffic, and integration into a SIEM or SOC workflow where the output would actually reach an analyst.
