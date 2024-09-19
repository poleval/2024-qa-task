# Reading Comprehension

## Introduction
Question Answering is an important and broad field of research within Natural Language Processing. Previous editions of PolEval included passage retrieval tasks, which are crucial in narrowing down the range of documents relevant to a human question, but only after including a reading comprehension system, can the whole process of answering a question be fully automated.

Classical systems for text comprehension relied on span extraction, but this is a limiting technique: it does not work well for morphologically rich languages (such as Polish) and does not fit well with answering yes-no questions. Recent advances in large, generative language models show that high zero-shot performance is easily achievable. However, aligning these models with more precise task definitions is still challenging, and traditional supervised learning paradigms still outperform GPT. Moreover, the problem of hallucinations is still pervasive, and so generative models will often prefer to fabricate answers instead of stating that the passage is irrelevant.


## Task Definition
The goal of this task is to develop a system for open-domain machine reading comprehension for the purposes of question answering. A system will be given a question with a paired passage. Some of the questions are "impossible", i.e. they are relevant to the passage, but the passage contains no answer. Others can be answered based on the passage and have gold answers listed. The gold answers **do not** have to be identical with any span from the text, though it usually is the case, and in a large majority of cases, are very close to some fragment from the text (with the notable exception of yes/no questions).

The participant will be given:

1. Training set consisting of 11 624 passages, each associated with a list of questions. In total, there are 56 618 questions in the train subset.
2. A dev set is supplied which can be used to evaluate the system, or as additional training data. It contains 1 453 paragraphs with 7 060 questions.

The system will be tested on two separate test sets (A and B). For each test pair (context + question), the system is supposed to generate an answer based on the information given in the context. Each model will be scored on two separate metrics:

1. Text similarity as assessed by Levenshtein edit distance, calculated for lowercased strings, normalized by length of the longer sequence, measured only on the answerable subset of the questions. This score measures the power of the system to answer questions.
2. Binary F1 score on the classification with respect to answerability. This score measures the capacity to recognise when then information is sufficient, and when to abstain from answering.


The two scores will be averaged with equal weights to generate the final score, which will be used to select the winner. The scores for each individual test set will be aggregated with weights equal to their proportion of the total number of examples.

Participants are free to use any publicly available datasets to develop their systems. It is forbidden to manually label the test examples.

## Dataset

All the subsets come from the same source (Polish Wikipedia), which was annotated as part of the CLARIN-BIZ initiative, to form the PoQuAD dataset (the Polish Question Answering Dataset). All subsets exhibit a similar distribution of data.

### Dataset format
The training data largely follow the SQuAD format, they are available [here](https://huggingface.co/datasets/clarin-pl/poquad/tree/main). The test dataset will be made available separately and will follow the same format. The dataset is divided into articles, and for each article there is **at least one** and up to two annotated paragraphs. Each paragraph can contain up to five questions. For instance, the following is a paragraph with one question:

**Update:** the test-A/test-B questions needed for the predictions were published on [github](https://github.com/poleval/2024-qa-task/tree/main) on 19th September.

```
{
 "id": 9773,
 "title": "Miszna",
 "summary": "Miszna (hebr. <200f>משנה<200e> miszna „nauczać”, „ustnie przekazywać”, „studiować”, „badać”, od <200f>שנה<200e> szana „powtarzać”, „różnić się”, „być odmiennym”; jid. Miszne) – w judaizmie uporządkowany zbiór tekstów ustnego prawa uzupełniający Torę (Prawo pisane). Według wierzeń judaizmu stanowi ustną, niespisaną część prawa nadanego przez Boga na Synaju, tzw. Torę ustną. Jest świętym tekstem judaizmu i jest traktowana na równi z Tanach (Biblią hebrajską). Zbiór był w Izraelu od wieków przekazywany ustnie z pokolenia na pokolenie, zwiększył swój rozmiar szczególnie w okresie od III w. p.n.e. do II w. n.e. w wyniku systematycznego uzupełniania komentarzy przez tannaitów, żydowskich nauczycieli prawa ustnego. Miszna została spisana dopiero w II–III w. Prace redakcyjne zapoczątkował rabin Akiba ben Josef, a kształt ostatecznej redakcji tekstu nadał Juda ha-Nasi. Miszna składa się z 6 porządków (hebr.: sedarim), które dzielą się na 63 traktaty, te zaś na rozdziały i lekcje. Miszna jest częścią Talmudu i zawiera podstawowe reguły postępowania i normy prawne judaizmu.",
 "url": "https://pl.wikipedia.org/wiki/Miszna",
 "paragraphs": [
   {
     "context": "Pisma rabiniczne – w tym Miszna – stanowią kompilację poglądów różnych rabinów na określony temat. Zgodnie z wierzeniami judaizmu Mojżesz otrzymał od Boga całą Torę, ale w dwóch częściach: jedną część w formie pisanej, a drugą część w formie ustnej. Miszna – jako Tora ustna – była traktowana nie tylko jako uzupełnienie Tory spisanej, ale również jako jej interpretacja i wyjaśnienie w konkretnych sytuacjach życiowych. Tym samym Miszna stanowiąca kodeks Prawa religijnego zaczęła równocześnie służyć za jego ustnie przekazywany podręcznik.",
     "qas": [
       {
         "question": "W jakich formach występowała Tora przekazana Mojżeszowi?",
         "answers": [
           {
             "text": "pisanej, a drugą część w formie ustnej",
             "answer_start": 210,
             "answer_end": 248,
             "generative_answer": "pisanej, ustnej"
           }
         ],
         "is_impossible": false
       }
     ]
   }
 }
```

The gold answer for the question is given by the "generative_answer" key. The "answer" key corresponds to the extractive answer and is **not** taken into account during the evaluation process. However, we decided to leave this as an additional layer of information which can be exploited for improving the systems.

In cases where the question is impossible, the annotation will look like this:

```
       {
         "question": "Kto napisał Torę?",
         "plausible_answers": [
           {
             "text": "Boga",
             "answer_start": 150,
             "answer_end": 154,
             "generative_answer": "Bóg"
           }
         ],
         "is_impossible": true
        }
```

The answers listed under the "plausible_answers" key are to be treated as distractors, **not** as gold answers.


### Downloading datasets
The train and dev sections of the dataset are to be found [here](https://huggingface.co/datasets/clarin-pl/poquad/tree/main).


## Evaluation
The predictions will be evaluated using the attached evaluation script, which calculates normalized Levenshtien and F1 and averages them.


### Submission format
The in.tsv file contains one question identifier per line, with each question identified by a unique ID in the following format:

<article_id>\_<paragraph_number>\_<question_number>

Note that paragraph and question numbers are 0-indexed.

Your submission should be written to the two out.tsv files (one in test-A, second in test-B), with each answer on a new line. The order of answers should match the order of questions in the in.tsv file. For example, the answer to the question on the 5th line of the in.tsv file should be on the 5th line of the out.tsv file.

A perfect solution should exactly match the contents of the expected.tsv file. To see examples of the input and expected output formats, refer to the in.tsv and expected.tsv files in the train and dev-0 folders.


## Baseline
A baseline based on GPT 3.5, with a paragraph from the dev set listed as examples, achieves the following scores:

|                  | nLev. | bin. F1 | SCORE |
|------------------|-------|---------|-------|
| GPT 3.5 few shot | 67.25 |  48.20  | 57.73 |
| plT5 baseline    | 83.25 |  57.67  | 70.46 |


## References
1. Pranav Rajpurkar, Jian Zhang, Konstantin Lopyrev, Percy Liang. 2016. [SQuAD: 100,000+ Questions for Machine Comprehension of Text](https://arxiv.org/abs/1606.05250)
2. Pranav Rajpurkar, Robin Jia, Percy Liang. 2018. [Know What You Don't Know: Unanswerable Questions for SQuAD](https://arxiv.org/abs/1806.03822)
3. Ryszard Tuora, Natalia Zawadzka-Paluektau, Cezary Klamra, Aleksandra Zwierzchowska, Łukasz Kobyliński. 2022. [Towards a Polish Question Answering Dataset (PoQuAD)](https://link.springer.com/chapter/10.1007/978-3-031-21756-2_16)

## Challenge metadata

Tags: poleval-2024, reading-comprehension, question-answering
