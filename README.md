# Testing Estonian spell-checking

This repository contains a gold standard test set of Estonian L2 learner texts and spelling correction output of various spell-checking tools available for Estonian. The work has been described in the following paper:
> Allkivi-Metsoja, K., & Kippar, J. (2023). [Spelling Correction for Estonian Learner Language](https://aclanthology.org/2023.nodalida-1.79/). _Proceedings of the 24th Nordic Conference on Computational Linguistics (NoDaLiDa)_. ACL Anthology, 782–788.

**Update:** The best Jamspell models have been tested together with an error-correction list used for word replacement as a preprocessing step. See the directory [`output_jamspell/with_correction_list`](output_jamspell/with_correction_list).

## Tools in comparison
L2 learners tend to make specific spelling errors, such as real-word errors, diacritic errors or pronunciation-induced errors that may have a large edit distance. Language-independent spell-checking algorithms that rely on n-gram models can offer a simple solution for improving learner error detection and correction due to context-sensitivity. Our aim was to evaluate the performance of bi- and trigram based statistical spelling correctors on an error-tagged set of A2–C1-level texts. For this purpose, we used:
* [Peter Norvig's algorithm](http://norvig.com/spell-correct.html) enhanced with a bigram language model in addition to a unigram model
* compound aware version of [Symmetric Delete Spelling Correction (Symspell)](https://github.com/wolfgarbe/SymSpell) with a bigram dictionary
* [Jamspell algorithm](https://github.com/bakwc/JamSpell) that additionally employs a trigram language model for selecting the highest-scored correction candidate

The training material came from the [Estonian National Corpus (ENC) 2019](https://metashare.ut.ee/repository/browse/estonian-national-corpus-2019/cd9633fab22e11eaa6e4fa163e9d4547b71a2df64d1f43f1ac26dbd8508ea951/). Jamspell and Norvig’s spelling corrector were trained on a random sample of 6 million sentences and over 82 million words retrieved from the Reference Corpus (RC) that represents the 'standard' varieties of Estonian – mostly newspaper texts but also fiction, science and legislation texts from 1990–2008. The sample constitutes nearly half of the RC; increasing the volume of the training set did not improve the correction results. Symspell, on the other hand, reached the best results with a uni- and bigram frequency dictionary based on the full ENC 2019 including over 1.5 billion words.

The newly trained spell-checking models were compared to existing correction tools:
* the only open-source spell-checker from the [Vabamorf library](https://github.com/Filosoft/vabamorf), which is lexicon- and rule-based
* MS Word speller (Microsoft 365)
* Google Docs spelling and grammar checker

## Test data and evaluation
The correction output was evaluated on a set of 84 error-annotated proficiency examination writings from the [Estonian Interlanguage Corpus (EIC)](https://elle.tlu.ee/tools/wordlist). Divided between four proficiency levels (A2, B1, B2, and C1), the texts contain 1,054 sentences, 9,186 words (excluding anonymized identifiers), and 309 spelling errors in total. We distinguished 263 simple spelling errors and 46 mixed errors, i.e., spelling 
mistakes co-occurring with another error such as word choice, inflected form, or capitalization error. 

The texts have been morphologically annotated in the CoNLL-U format, using the Stanza toolkit, and manually error-tagged, indicating the error type, scope, and correction in the field for miscellaneous token attributes ('MISC'). While the custom tagset denotes various orthographic and grammatical errors, we only rated the correction of words with a spelling error (`Error=Spelling`). However, we did not count a system edit as unnecessary if another word-level error, such as word form, word choice, capitalization or missing whitespace error, was annotated. The annotation format allows for several corrections per token but is limited to one error annotation per sentence. This has no significant effect on the analysis of spelling errors, which occur regardless of the sentence structure.

We evaluated spelling error detection and correction based on precision, recall and F0.5 score. In error correction, we prioritized the speller’s accuracy of defining the best correction and thus focused on the highest-ranked suggestion. The cases of mixed errors were reviewed manually to find partial corrections fixing only the spelling of an otherwise erroneous word (e.g., _parnu_ ~ _pärnu_ instead of _Pärnu_, which is a town name and should be capitalized). Both full and partial word corrections were considered in calculating the evaluation metrics. The comparison is summarized in table 1.

**Table 1.** Spelling error detection (ED) and correction (EC) metrics (%)

| Speller | ED recall | ED precision | ED F0.5 | EC recall | EC precision | EC F0.5 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| Jamspell | 67.0 | **89.6** | 83.9 | 50.8 | 68.0 | 63.7 |
| Norvig | 62.8 | 84.3 | 78.9 | 43.0 | 57.8 | 54.1 |
| Symspell | 38.5 | 86.2 | 69.1 | 17.5 | 39.1 | 31.4 |
| Google | **69.6** | 78.8 | 76.7 | **61.2** | **69.2** | **67.5** |
| MS Word | **69.6** | 87.8 | 83.4 | 42.7 | 53.9 | 51.2 |
| Vabamorf | 69.3 | 89.2 | **84.3** | 35.0 | 45.0 | 42.6 |

## Improvements and final results
Jamspell as the best-performing statistical corrector was further trained on various ENC datasets to analyse their effect on the correction results. The highest error detection and correction precision were achieved by the model trained on the Estonian Web Corpus 2019 (Web19) that comprises a diverse selection of texts, from informal blog posts and forum discussions to periodicals and educational materials. This model was the least likely to make unnecessary corrections but also to detect spelling errors, thus having the lowest recall. The initial model trained on the RC sample still scored highest in error detection and correction recall, being able to identify and correct the largest amount of spelling errors.

Interestingly, models trained on samples of the RC and other more 'standardized' subcorpora (Wikipedia, DOAJ) achieved better or similar results compared to the models trained on full subcorpora. Contrary to that, the model trained on the whole Web19 performed better than the models based on web samples. We also merged the RC and Web19 material in an equal ratio and in a ratio of 10:1. The latter model offered a compromise in the precision-recall trade-off, yielding a higher precision than the RC model and a higher recall than the Web19 model.

The three highest-ranked Jamspell models were combined with a list of reoccurring learner errors and expert corrections. The list was compiled based on the EIC data  (excluding the test material) and contains appr. 3,000 spelling errors that occur in multiple L2 texts, minimally 3 times in total. Only errors that could be assigned a single correction were included. Using list-based word replacements as a preprocessing step significantly improved the error detection and correction scores, outperforming all other tested tools (Google has a slightly better error correction recall but a lower precision and F-score). The results are presented in table 2.

**Table 2.** Spelling error detection (ED) and correction (EC) metrics (%) of Jamspell models

| Speller | ED precision | ED recall | ED F0.5 | EC precision | EC recall | EC F0.5 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| RC sample | **67.0** | 89.6 | **83.9** | **50.8** | 68.0 | 63.7 |
| Web19 | 53.7 | **94.3** | 81.9 | 42.4 | **74.4** | **64.7** |
| RC + Web19 | 60.2 | 91.2 | 82.7 | 46.0 | 69.6 | 63.1 |
| RC sample with errorlist | **73.5** | 90.8 | **86.7** | **59.9** | 74.0 | 70.7 |
| Web19 with errorlist | 64.4 | **94.8** | 86.6 | 53.7 | **79.0** | **72.2** |
| RC + Web19 with errorlist | 69.9 | 92.3 | **86.7** | 56.6 | 74.8 | 70.3 |

### Notes
* Three best Jamspell models and the error-correction list have been made available in [HuggingFace](https://huggingface.co/Jaagup) and are implemented in the [spelling and grammar correction toolkit](https://koodivaramu.eesti.ee/tartunlp/corrector) for Estonian. 
* The test data forms a subset of a larger error-annotated corpus, which has now been converted to the M2 annotation format and can be found [here](https://github.com/tlu-dt-nlp/EstGEC-L2-Corpus).
