# Atrial Fibrillation (AFib) binary classifier
這個專案的目標在於實作一個簡易的、能夠自動偵測心房纖維性顫動(Atrial Fibrillation)的二元分類器，好協助居家照護者或病友能夠及早發現心律異常，在有限時間內做出正確的醫療決策。 
# Abstract:

這個專案引用了這篇論文[1]的特徵工程方法，並根據實際情況做出了一定幅度的修改，目的在於增加分類器的穩健性(Robustness)及可解釋性(Interpretability)。

演算流程:
1. 將一段30分鐘的ECG做預處理、透過neurokit內建的peak dection algorithm取得它的心律變異性(Heart rate variability, HRV)。
2. 將HRV畫成Poincare plot可以看見清晰的分群。
3. 使用DBSCAN分群算法計算分群數量(群的尺度大小參考自[2])，另外再計算Poincare plot的統計量，獲得4 dimensions featue values。
4. 透過CatBoostClassfier 對 featue values做learning。 

# Dataset
我在這裡使用PAF Prediction Challenge Database ，這是一個由Computing in Cardiology conference 為了其2001發起的挑戰而構建的database，這個挑戰的目標是開發預測陣發性心房顫動 (PAF) 的自動化方法。

該database分為學習集（名稱格式為 n *和 p *的記錄）和測試集（名稱格式為 t *的記錄）。此database中含有100組30分鐘的含標籤的ECG片段，一半包含AFib訊號(p)，一半是正常訊號(n)。
另外還有不含標籤的測試集紀錄以及AFib發作後5分鐘的ECG訊號(以c做開頭)，這裡我們不會用到。

若想知道更詳細的資訊可至physio.net 的官網了解 

PAF Prediction Challenge Database: https://www.physionet.org/content/afpdb/1.0.0/

# Pictures for demonstration

## Dataset distribution
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/data%20balance.png)
## Sanity check of peak dection algorithm
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/n10_%E6%AD%A3%E5%B8%B8%E6%83%85%E5%BD%A2_ECG.png)
## Poincare of a normal person
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/poincare_plot_normal.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/poincare_plot_normal_colored.png)

## Distribution of number of clusters of normal people's Poincare plot
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/Clusters%20of%20normal.png)

## Distribution of number of clusters of AFib patients' Poincare plot 
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/Cluster%20of%20AFib%20patients.png)

## Poincare of an AFib patient
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/poincare_plot_AFib.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/poincare_plot_AFib_colored.png)
## Performance of SVM on feature values 
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/SVM%20cm.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/SVM%20learning%20curve.png)
## Performance of CatBoostClassifier on feature values
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/CatBoost_Learning_Curve.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/confusion_matrix.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/Non_normalized_cm_catbosst.png)
# Result
Let AFib = Positive, Normal = Negative.

Precision : 92%.

Recall : 80%.

F1-score : 85.5%.

![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/confusion_matrix.png)


# Reference
[1] Park, J., Lee, S., & Jeon, M. (2009). Atrial fibrillation detection by heart rate variability in Poincare plot. Biomedical engineering online, 8(1), 1-12.

[2] Käsmacher, H., Wiese, S., & Lahl, M. (2000). Monitoring the complexity of ventricular response in atrial fibrillation. Discrete Dynamics In Nature And Society, 4, 63-75.
