# Machine Failure Prediction with PySpark

A small project that uses machine sensor readings to predict which machines are likely to fail.

## The problem

In a factory, an unexpected machine failure can stop a production line. If you can spot the warning signs early, you can fix a machine before it breaks. This project tests whether sensor readings can give that early warning.

## The data

I used the AI4I 2020 Predictive Maintenance dataset (from the UCI Machine Learning Repository). It has 10,000 machine records. Each record includes:

- Machine type (low, medium, or high quality)
- Air temperature and process temperature
- Rotation speed
- Torque (twisting force)
- Tool wear (minutes of use)
- Whether the machine failed

This is a simulated dataset made to look like real factory data. Only about 3.5% of machines fail, which makes the problem harder.

## What I did

1. **Loaded and cleaned the data** using PySpark. I renamed columns, checked for missing values and duplicates, and added one new column: the gap between process and air temperature.
2. **Asked questions with Spark SQL**, such as which machine type fails most often and how failed machines differ from working ones.
3. **Trained a random forest model** with Spark MLlib to predict failures. I trained on 80% of the data and tested on the other 20%.
4. **Fixed a problem with rare failures.** My first model scored 97.7% accuracy, but that number is misleading. Since almost every machine works fine, a model that always guessed "no failure" would score about 96%. The model only caught 26 of the 70 real failures in the test set. So I told the model that missing a failure costs more than a false alarm, and tested three settings.

## Results

Test set: 2,000 or so machines, with 70 real failures.

| Setting | Failures caught (of 70) | False alarms | Alerts that were real |
|---|---|---|---|
| No weighting (first model) | 26 (37%) | not recorded | not recorded |
| Weight 5 | 42 (60%) | 12 | 78% |
| Weight 10 | 47 (67%) | 38 | 55% |
| Weight 20 | 61 (87%) | 105 | 37% |

I chose weight 5 as the main result because it catches most failures while keeping about 4 in 5 alerts correct. The best setting in a real factory depends on what a missed failure costs compared to an unneeded check. If a breakdown is very expensive, a higher weight could make more sense.

**Strongest warning signs** (in order): rotation speed, torque, the temperature gap, and tool wear.

## What I learned

- Accuracy alone can hide a weak model when one outcome is rare. Checking how many real failures the model caught gave a much clearer picture.
- Catching more failures always means more false alarms. Choosing the balance is a business decision, not just a technical one.

## Limits

- The test set has only 70 failures, so the numbers are rough. A small change in results could be luck.
- The data is simulated, so results may not match a real factory.
- I tested on one split of the data only.

## Next steps

- Test on several splits of the data (cross-validation) for steadier results
- Try other models and compare them
- Save the cleaned data to a database as part of a full data pipeline

## How to run it

1. Open the notebook in Google Colab or Kaggle.
2. Run the cells in order. The first cell installs PySpark and downloads the dataset.

**Tools used:** Python, PySpark, Spark SQL, Spark MLlib
