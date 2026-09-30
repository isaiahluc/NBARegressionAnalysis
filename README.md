# NBA Regression Analysis

# What predicts NBA win percentage?
A multiple linear regression study of team win percentage across two NBA seasons (2015-16 and 2022-23), built in R. Group project for STAT 3220 at the University of Virginia.
# Research questions
Do teams with an older average roster win less?

Do offensive and defensive rating drive win percentage, and which matters more?

Does starting a season with a new head coach lower win percentage?
# Data
60 team-seasons (all 30 teams in 2015-16 and 2022-23). The two seasons are seven years apart, so rosters, coaches, and front offices had turned over enough to treat each team-season as a separate observation.


<img width="1092" height="532" alt="image" src="https://github.com/user-attachments/assets/a688805b-b222-44b9-be4a-c227c1cd7cdd" />

Variable
Description
WRate (response)
Share of regular-season games won
Age
Average age of active players
MOV
Average margin of victory (points)
ORtg / DRtg
Points scored / allowed per 100 possessions
Pace
Possessions per 48 minutes
Att, Att_G
Total and per-game home attendance
Conf, POYP, NewCoach
Conference, made playoffs prior year, new head coach (yes/no)


Sources: Basketball Reference (team stats, coaching changes, playoff teams) and TeamRankings (win percentage).
# Method
Exploratory analysis. The response was roughly normal, so no transformation was needed. Coach status and conference showed little separation in win percentage.
Multicollinearity check. Wins and losses were dropped because they define the response. The full model still had VIFs above 1,000 for MOV, ORtg, and DRtg, and above 100 for the two attendance measures.
Stepwise selection (entry/exit threshold p = 0.15, confirmed at 0.20) kept only MOV and Age. VIFs in the reduced model were 1.22.
Diagnostics. Residual plots showed no curvature or fanning, and the Q-Q plot was close to normal. Cook's distance and deleted studentized residuals flagged observations 10, 47, and 57 as influential.
Ridge regression as a comparison method (R² = 0.946 vs. 0.947 for OLS).
# Results
Final model (n = 60, adjusted R² = 0.947):

WRate = 0.312 + 0.0297 * MOV + 0.0071 * Age

Term
Estimate
95% CI
p-value
MOV
0.0297
[0.0276, 0.0318]
< 0.001
Age
0.0071
[0.0022, 0.0120]
0.005


Holding the other variable constant, each extra point of average margin adds about 3 percentage points of win percentage, and each extra year of average roster age adds about 0.7 points.

Checked against 2024-25 results:

Team
Predicted
Actual
Error (pts)
Oklahoma City
.871
.829
4.2
Cleveland
.785
.780
0.5
Denver
.620
.610
1.0

# Answers to the research questions
Age: No. Older rosters won more, the opposite of our hypothesis. Experience appears to outweigh physical decline at the team level within the observed range (ages 22 to 31).
Offensive vs. defensive rating: The model can't separate them. MOV is essentially offensive rating minus defensive rating scaled by pace, which is why all three had VIFs above 1,000. Once MOV is in the model, the ratings add nothing new.
New head coach: No significant effect in this sample.
# Limitations
MOV is close to a restatement of winning (r = 0.97 with WRate). The high R² mostly reflects that teams that outscore opponents win games. That makes the model accurate but not very useful as a forecast, since MOV isn't known until the season is played. The 2024-25 check used same-season MOV, so it tests fit, not true prediction.
Small sample. 60 observations from two seasons limits power, especially for the categorical variables.
Influential points were kept. Refitting without observations 10, 47, and 57 would show how sensitive the Age coefficient is.
The validation teams were all strong ones, not a random draw.

A natural next step is to predict win percentage from prior-season MOV, age, and roster changes, which would make it a real forecasting model.
Reproducing
data <- read.csv("Workable Stat 3220 Project Data.csv", header = TRUE)

data <- data[-c(61, 62), ]   # drop summary rows

data <- data[, -c(15:31)]    # drop unused columns

model <- lm(WRate ~ MOV + Age, data = data)

summary(model)

confint(model)

Required packages: car (VIFs), olsrr (stepwise selection, diagnostics), glmnet (ridge), corrplot, ggplot2.
