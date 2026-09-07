# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Amanda Padua
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** flyrank-ml-internship-starter
- **Date:** September 7, 2026

## 0. Abstract

Content teams usually have a lot of pages to manage, so it can be hard to know which ones should be reviewed first. In this project, I looked at recent search performance and other content-related signals from the FlyRank internship dataset to find pages that may need attention. I built a Logistic Regression model and tested it on pages from six clients that were kept separate from the training data, and the model achieved a Precision@20, Precision@50, and Precision@100 of 1.00 on the test set. I then used the model scores to rank the pages and added simple priority levels and reasons to make the results easier to understand. Overall, I see the model as a useful starting point for helping content teams decide which pages to review first, but it does not guarantee that making changes to a page will improve its future performance.

## 1. Problem framing

The decision I want to support is which content pages a content team should review first. The unit of analysis is an individual content page, and the output is an opportunity score that I use to rank the pages and give them a simple priority level.

A FlyRank editor could use the ranking to start with the highest-priority pages and then manually check things such as whether the content is still up to date, matches search intent, has a low CTR, or has a weaker average position. The model is not deciding what should be changed; it is helping narrow down where the review can start.

A wrong recommendation can waste time if a page is flagged even though it does not really need attention. On the other hand, if a page that needs attention is ranked too low, the team may not review it as early as it should.

I used machine learning because there are many pages and several different signals to look at at the same time. A Logistic Regression model gives me a consistent way to combine these signals and rank the pages instead of relying only on manual checking.

## 2. Data safety

For this project, I used the FlyRank ML Internship dataset. I mainly worked with search performance and content-related information for each page. The prepared dataset had 30,000 pages and 52 columns.

I left out some columns from the model because they were not needed or could cause leakage. I removed `content_id` and `client_id` because they are identifiers, not useful signals for deciding if a page needs attention. I still used `client_id` to split the data by client so that the same client would not appear in both the training and test sets.

I also left out `trend_direction`, `trend_pct`, and `is_declining_label` from the model. These fields are connected to the target, so using them could give the model information about the answer.

I checked the feature preparation code and confirmed that `is_declining_label` is created from `trend_direction`. Because of this, I kept these fields out of the model features.

I also made sure the notebook and report do not include private client information such as client names, domains, URLs, private search queries, credentials, or raw data exports.

Overall, I tried to keep the model focused on the actual page and performance signals while keeping identifiers and target-related information separate.

## 3. Baseline

For my baseline, I used the approach I built in Week 4 as a starting point. The Week 4 baseline had a Precision@50 of 0.40.

I used Precision@50 because I wanted to see how many of the top 50 pages ranked by the method were actually pages that were marked as declining. This fits the main goal of my project, which is to help a content team decide which pages to review first.

I compared the baseline and the Logistic Regression model using the same evaluation setup so that the comparison would be fair. The baseline had a Precision@50 of 0.40, while my model got 1.00.

This is an improvement of 0.60 precision points. In other words, the model did a better job than the baseline at putting declining pages near the top of the list.

The baseline gave me a useful starting point because it helped me see whether using a machine learning model actually improved the ranking instead of just looking at the model's results on their own.

## 4. Model / analysis

For my model, I used Logistic Regression. I chose it because it is a fairly simple model and it allowed me to look at different page and performance signals together. The model then gave each page a score that I could use to rank which pages might need attention first.

The model used these 47 features:

`search_volume`, `competition`, `competition_level`, `cpc`, `content_type`, `main_intent`, `word_count`, `char_count`, `provider_used`, `model_used`, `impressions_90d`, `clicks_90d`, `pageviews_90d`, `sessions_90d`, `users_90d`, `engaged_sessions_90d`, `ai_sessions_90d`, `scroll_events_90d`, `days_with_impressions`, `days_with_sessions`, `impressions_last_30d`, `clicks_last_30d`, `sessions_last_30d`, `impressions_prev_30d`, `clicks_prev_30d`, `sessions_prev_30d`, `content_age_days`, `age_tier`, `age_tier_order`, `days_since_last_update`, `freshness_tier`, `word_count_tier`, `char_count_tier`, `ctr`, `avg_position`, `engagement_rate`, `scroll_rate`, `ai_traffic_pct`, `impression_tier`, `position_tier`, `log_impressions_90d`, `log_clicks_90d`, `log_sessions_90d`, `log_ai_sessions_90d`, `has_clicks`, `has_ai_sessions`, and `measurable_opportunity`.

I did not use `content_id` or `client_id` as features because they are just identifiers. I also left out `trend_direction`, `trend_pct`, and `is_declining_label` because they are connected to the target and could give the model information about the answer.

For the target, I used `trend_direction == "down"` to identify pages that were showing a decline. This was then used to create the `is_declining_label` for the model.

After training the model, I used the probability it gave each page as the opportunity score. I then ranked the test pages from the highest score to the lowest score. This gave me a simple way to identify which pages could be reviewed first.

## 5. Evaluation

I split the data by client so that the same client would not appear in both the training and test data. I used 26 clients for training and 6 different clients for testing. This gave me 27,675 pages for training and 2,325 pages for testing.

Out of the 2,325 test pages, 909 were labeled as declining. This means that about 39.1% of the test pages were declining.

I used Precision@20, Precision@50, and Precision@100 to check how well the model ranked the pages that may need attention. The model got 1.00 for all three metrics. This means that all of the pages in the top 20, top 50, and top 100 were labeled as declining in the test set.

I also compared the model with my Week 4 baseline. The baseline had a Precision@50 of 0.40, while my model got 1.00. This gives an improvement of 0.60 precision points.

When I looked at the predictions overall, the model correctly classified 1,838 out of the 2,325 test pages. It predicted 654 pages as positive, while 909 pages were actually labeled as declining.

The results were very strong, but I would still be careful about treating them as a final answer. The test set only included six held-out clients, so the results could be different with another group of clients or with new data. I see the model mainly as a tool to help rank pages for review, rather than something that guarantees which pages will improve.

## 6. Interpretation

After looking at the results, I noticed that recent performance decline was the main pattern in the pages ranked highest by the model. In the top 100 pages, 66 had a recent performance decline, 33 had a recent decline along with a low CTR, and 1 had a recent decline together with a weaker average position.

I also looked at some of the features used by the model. Signals related to impressions, measurable opportunity, content age, word count, and recent performance appeared to be useful for the model. This showed me that the model was looking at different types of information instead of depending on just one signal.

One thing I found interesting was that content age and some of the performance signals had different relationships with the target. This reminded me that page performance can be affected by several signals at the same time.

The model also performed very well on the test set, with a Precision@20, Precision@50, and Precision@100 of 1.00. However, I would not assume that these results will always be the same with new clients or new data. The results came from the specific test set I used.

Overall, the model helped me find a group of pages that could be worth reviewing. The biggest pattern I saw was recent performance decline, but I would still use the model as a starting point and have a person review the pages before making any decisions.

## 7. Recommendation

I would use the opportunity score to help decide which pages should be checked first. Instead of going through every page one by one, a content editor could start with the pages that have the highest scores.

For the high-priority pages, the editor could then check things like whether the content is still up to date, matches the search intent, has a low CTR, or has a weaker average position. After looking at the page, they can decide if it needs to be updated, monitored, or left as it is.

I think this can make the content review process easier because the model gives the team a shorter list to start with. The priority levels and reasons also make the results easier to understand instead of only showing a score.

I feel more confident using the model to help rank pages than using it to make the final decision. The results were strong on my test set, but they could be different when the model is used with other clients or new data.

A high opportunity score does not mean that changing the page will definitely improve its performance. It only means that the page has signals that match the type of pages the model identified as declining.

Overall, I see this model as a helpful starting point for a content team. It can help them decide where to look first, while the final decision should still be made by a person who reviews the page and its context.

## 8. Reproducibility

I used Python, pandas, scikit-learn, and DuckDB to complete this project. I followed the project structure provided by FlyRank and used the feature preparation script to get the data ready for the model.

The main steps I followed were:

1. Cloned the FlyRank internship repository.
2. Ran the feature preparation script to create the prepared dataset.
3. Loaded the prepared data into the notebook.
4. Split the pages by client so that the test clients were kept separate from the training clients.
5. Removed the ID and target-related columns before training the model.
6. Trained the Logistic Regression model.
7. Used the model to create an opportunity score for each test page.
8. Checked the results using Precision@20, Precision@50, and Precision@100.
9. Ranked the pages and added priority levels and simple reasons.

I used a random seed of `42` so that the client split and model results could be reproduced.

The prepared dataset had 30,000 rows and 52 columns. After the client split, I had 27,675 pages for training and 2,325 pages for testing.

The main code for preparing the data, training the model, checking the results, and creating the recommendations is included in the notebook. This means the steps can be followed again if the project needs to be rerun.

If the data or environment changes, I would rerun the analysis rather than assuming that the same results will always be produced.

## 9. Acknowledgments & data credit

I completed this project as part of the FlyRank ML Internship. I used the dataset and the materials provided during the internship to build and test my project.

I’m thankful to FlyRank AI for providing the dataset and giving me the opportunity to work on this project. It helped me put what I learned during the internship into practice.

**Data credit:** Built on the FlyRank ML Internship dataset, provided by [FlyRank AI](https://flyrank.ai).
