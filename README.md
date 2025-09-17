# Решение для отборочного задание на смену Sirius T-bank (Ильин Глеб ИТМО)

Начну с ответа на вопрос *"Часто при разработке и оценке рекомендательных систем такие айтемы почему-то (кстати, как думаете, почему?)* игнорируются." все из за явления которое называется feedback loop, когда алгоритм предлагает товары с которыми больше всего взаимодействует, в таком случае непопулярные товары не имеют шанса на продвижение.



*"К задаче рекомендаций можно подойти как к задаче: Регрессии, Классификации, Ранжирования?"*
Также по моему мнению здесь будет задача ранжирования тк из за проблемы холодного старта и обилия холодных айтемов мы будем мерить схожесть (и соответственно ранжировать по схожести) все товары в тч холодные и те товары которые нравятся юзерам

## Этапы работы по порядку от EDA до Summary
## EDA: [baseline_solution/eda.ipynb](https://github.com/GlebIsrailevich/t_bank_sirius25/blob/master/baseline_solution/eda.ipynb)
В EDA мы видим что в интеракциях есть Explicit (явный фидбек) в виде рейтинга книг. Также у нас есть фича is_read но кмк это уже implicit feedback тк то что книга прочитана еще не означает что она понравилась юзеру (тем более у нас распределение 60 на 40). 
Если бы мы использовали фичу is_read то мы бы решали задачу классификации, но поскольку лучше брать фичу ratings то будем решать задачу регрессии на рейтинг от 0 до 5.

Еще в EDA я рассмотрел таймстемпы (спойлер особо никаких закономерностей нет). Также при рассмотрении популярности как айтемов, так и активности юзеров видим что у распределений тяжелый хвост.


## Векторизуем текстовые фичи [baseline_solution/text_encoder.ipynb](https://github.com/GlebIsrailevich/t_bank_sirius25/blob/master/baseline_solution/text_encoder.ipynb)
Так как весь текст представлен на английском языке берем мощный текстовый эмбеддер для английского языка (BGE-M3)[https://huggingface.co/BAAI/bge-m3]. Получаем вектора длиной 1024 для фичей title и description

## Векторизуем картиночные фичи  [baseline_solution/visual_encoder.ipynb](https://github.com/GlebIsrailevich/t_bank_sirius25/blob/master/baseline_solution/visual_encoder.ipynb)
Используем визуальный энкодер (CLIP)[https://huggingface.co/openai/clip-vit-base-patch32] 
Также пробовал (dinov3)[https://huggingface.co/facebook/dinov3-convnext-small-pretrain-lvd1689m] но эмбеддинги CLIP работают лучше


## Multiarmed Bandits [methods/greedy_mab.ipynb](https://github.com/GlebIsrailevich/t_bank_sirius25/blob/master/methods/greedy_mab.ipynb)
Здесь реализован Epsilon-Greedy MAB, пытался также сделать UCB и Thompson sampling, но они дают меньшие метрики по NDCG и Recall но Coverage тоже остается высоким.
[Bandits](https://towardsdatascience.com/handling-feedback-loops-in-recommender-systems-deep-bayesian-bandits-e83f34e2566a/)
[Многорукие бандиты в Яндекс Лавке](https://www.youtube.com/watch?v=T0eRQiQSyMY&ab_channel=ODSAIRu)
Реализован алгоритм Multi Armed Bandit Epsilon-Greedy. Модель отлично предлагает холодные и непопулярные айтемы Метрика Coverage Близка к 1му, но NDCG и Recall хуже чем у ItemKNN.

## Catboost methods/catboost_pipeline.ipynb [methods/catboost_pipeline.ipynb](https://github.com/GlebIsrailevich/t_bank_sirius25/blob/master/methods/catboost_pipeline.ipynb)
Здесь полученные эмбеддинги я попробовал подать в CatboostRanker для формирования ContentBased рекомендаций (неудачный эксперимент, на имеющихся фичах катбуст выдает только теплые)

## methods/user_portrait.ipynb []()
Здесь я пытался сконкатенировать описания и названия прочитанных юзером книг и 

## KNN вариации [methods/knn_variations.ipynb](https://github.com/GlebIsrailevich/t_bank_sirius25/blob/master/methods/knn_variations.ipynb)
Здесь я пробую использовать разные метрики (косинус) и разные эмбеддинги по отдельности

## HNSW [methods/hnsw_test.ipynb]()
Здесь я совмещаю эмбеддинги тэгов, изображений описаний и названий книг 


## Summary or Ablation Study [methods/summary.ipynb](https://github.com/GlebIsrailevich/t_bank_sirius25/blob/master/methods/ablation_study.ipynb)
Итоги по задаче что получилось, а что нет + метрики

## Future work

- Из простого для каждого юзера конкатенировать описания прочитанных им книг (или просто тайтлов), далее получить эмбеддинг этого текста, это будет вектор предпочтений юзера. Этот эмбеддинг уже можно по метрике сравнивать с эмбеддингами описаний или названий книг

- При проблеме холодного старта у айтемов сразу подумал про Semantic IDs, времени их реализовать к сожалению не было, но если бы время было начал бы строить их с помощью RQ-VAE аналагично тому как представлено в статье [Better Generalization with Semantic IDs: A Case Study in Ranking for Recommendations](https://arxiv.org/abs/2306.08121)
Но еще бы я попробовал сделать SIDs на кластеризации как тут [«Semantic IDs: архитектура и наш опыт внедрения» — Александр Тришин WB](https://www.youtube.com/watch?v=kfiG2ChpM34&t=1473s&ab_channel=WBTech)

- Еще во время отбора я вдохновлялся статьей [Сама статья](https://arxiv.org/abs/2502.18965), [Тех репорт 1](https://arxiv.org/abs/2506.13695), [Тех репорт 2](https://arxiv.org/abs/2508.20900). Крутой момент то что мы избавляемся от этапа ранжирования где тот же самый бустинг может попасть в feedback loop и убрать наши новые непопулярные айтемы. Но трансформер у меня точно не заведется, Даже бустинг с эмбеддингами не очень быстро считался