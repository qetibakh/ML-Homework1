# House Price Prediction - Ames Housing Dataset

## პროექტის მიმოხილვა
- Kaggle-ის კონკურსი "House Prices: Advanced Regression Techniques" გულისხმობს სახლების ფასების პროგნოზირებას Ames, Iowa-ში 79 მახასიათებლის საფუძველზე.
- ჩემი მიდგომა მოიცავს დეტალურ cleaning-ს, feature engineering-ს, feature selection-სა და რამოდენიმე რეგრესიული მოდელის გამოცდას საუკეთესო შედეგის მისაღებად.

## რეპოზიტორიის სტრუქტურა
- `model_experiment.ipynb` - ძირითადი ნოუთბუქი cleaning, feature engineering, feature selection და ტრენირებისთვის
- `model_inference.ipynb` - საუკეთესო მოდელის გამოყენებით პროგნოზების გენერირება
- `data/` - მონაცემები
- `README.md` - პროექტის დოკუმენტაცია

## feature engineering
- კატეგორიული ცვლადების რიცხვითში გადაყვანა:
  - Ordinal Encoding გამოვიყენე ხარისხისა და მდგომარეობის მახასიათებლებისთვის (Ex, Gd, TA, Fa, Po)
  - One-Hot Encoding გამოვიყენე სხვა კატეგორიული მახასიათებლებისთვის

- NaN მნიშვნელობების დამუშავება:
  - რიცხვითი მახასიათებლები შევავსე მედიანით
  - კატეგორიული მახასიათებლები შევავსე მოდით
  - სპეციალური შემთხვევები (გარაჟი, სარდაფი) შევავსე "None"-ით
  - 11 რიცხვითი და 23 კატეგორიული მახასიათებელი აღმოჩნდა დაკარგული მნიშვნელობებით

- Cleaning მიდგომები:
  - ასიმეტრიული მახასიათებლების ლოგარითმული ტრანსფორმაცია (26 მაღალი ასიმეტრიის მქონე მახასიათებელი)
  - ახალი მახასიათებლების შექმნა (TotalSF, TotalBath, Age, Remodeled, RemodAge)
  - მნიშვნელოვანი თარიღების კომბინირება (Age, RemodAge)

## მახასიათებლების შერჩევა
- გამოყენებული მიდგომები:
  - კორელაცია სამიზნე ცვლადთან (SalePrice) - გამოვლინდა მაღალი კორელაციის მქონე მახასიათებლები სხვადასხვა ზღვრებზე (0.1, 0.2, 0.3)
  - Lasso რეგრესია - ავტომატურად შეირჩა 23 მნიშვნელოვანი მახასიათებელი
  - Random Forest მახასიათებლების მნიშვნელობა - შეირჩა ყველაზე მნიშვნელოვანი 30 და 50 მახასიათებელი

- შეფასება:
  - შევქმენი რამდენიმე მახასიათებლების ნაკრები (all_features_numeric, corr_0.1, corr_0.3, lasso_selected, rf_top30, rf_top50)
  - თითოეული ნაკრები შევაფასე cross-validation RMSE-ით
  - საუკეთესო შედეგი აჩვენა 'rf_top50' ნაკრებმა

## ტრენირება
- ტესტირებული მოდელები:
  - Linear Regression (საბაზისო მოდელი)
  - Random Forest (სხვადასხვა პარამეტრებით)
  - XGBoost (სხვადასხვა პარამეტრებით)

- Hyperparameter ოპტიმიზაციის მიდგომა:
  - Random Forest-ისთვის გამოვცადე პარამეტრები:
    - n_estimators: [50, 100, 200]
    - max_depth: [None, 10, 20, 30]
  - XGBoost-ისთვის გამოვცადე პარამეტრები:
    - learning_rate: [0.01, 0.05, 0.1]
    - max_depth: [3, 5, 7]
    - n_estimators: [100, 200]
  - GridSearchCV გამოვიყენე XGBoost-ის საბოლოო ოპტიმიზაციისთვის დამატებითი პარამეტრებით:
    - min_child_weight: [1, 3, 5]
    - subsample: [0.6, 0.8, 1.0]
    - colsample_bytree: [0.6, 0.8, 1.0]
  - Cross-validation (5-fold) გამოვიყენე ყველა მოდელის შესაფასებლად

- Overfitting ანალიზი:
  - ყველა მოდელისთვის გავზომე overfitting ratio (train_rmse / cv_rmse)
  - Linear Regression: ~0.94 (დაბალი overfitting)
  - Random Forest: ~0.37 (საშუალო overfitting)
  - XGBoost depth=7: ~0.05 (ძალიან მაღალი overfitting)
  - XGBoost depth=3: ~0.55 (ზომიერი overfitting)

- საბოლოო მოდელის შერჩევის დასაბუთება:
  - XGBoost აჩვენებს საუკეთესო შედეგებს cross-validation-ზე
  - მოდელი კარგად მუშაობს რთულ, არაწრფივ დამოკიდებულებებზე
  - ოპტიმალური პარამეტრები უზრუნველყოფს კარგ ბალანსს სიზუსტესა და overfitting-ს შორის

## MLflow Tracking
- MLflow ექსპერიმენტების ბმული: https://dagshub.com/qetibakh/ML-Homework1.mlflow
- ჩაწერილი მეტრიკები:
  - RMSE (Root Mean Squared Error) - მოდელის შეფასებისთვის
  - train_rmse და overfitting_ratio - overfitting-ის გასაზომად
  - მახასიათებლების რაოდენობა სხვადასხვა ეტაპზე
  - დაკარგული მნიშვნელობების რაოდენობა წმენდის შემდეგ

- საუკეთესო მოდელის შედეგები:
  - XGBoost მოდელი, რომელიც გამოიყენებს 'rf_top50' მახასიათებლების ნაკრებს
  - საუკეთესო პარამეტრები: learning_rate=0.05, max_depth=5, n_estimators=200, colsample_bytree=1.0, min_child_weight=1, subsample=0.6
  - RMSE: 0.1320
  - overfitting_ratio: 0.4164
  - Kaggle-ის საჯარო შეფასება: 0.13665