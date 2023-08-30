# Atrial Fibrillation (AFib) binary classifier
這個專案的目標在於實作一個簡易的、能夠自動偵測心房纖維性顫動(Atrial Fibrillation)的二元分類器，好協助居家照護者或病友能夠及早發現心律異常，在有限時間內做出正確的醫療決策。 
# Abstract:

這個專案引用了這篇論文[1]的特徵工程方法，並根據實際情況做出了一定幅度的修改，目的在於增加分類器的穩健性(Robustness)及可解釋性(Interpretability)。

演算流程:
1. 將一段30分鐘的ECG做預處哩、透過neurokit內建的peak dection algorithm取得它的心律變異性(Heart rate variability, HRV)。
2. 將HRV畫成Poincare plot可以看見清晰的分群。
3. 使用DBSCAN分群算法計算分群數量(群的尺度大小參考自[2])，另外再計算Poincare plot的統計量，獲得4 dimensions featue values。
4. 透過CatBoostClassfier 對 featue values做learning。 

# 結果
Let AFib = Positive, Normal = Negative.

Precision : 92%.

Recall : 80%.

F1-score : 85.5%.

[1] Park, J., Lee, S., & Jeon, M. (2009). Atrial fibrillation detection by heart rate variability in Poincare plot. Biomedical engineering online, 8(1), 1-12.

[2] Käsmacher, H., Wiese, S., & Lahl, M. (2000). Monitoring the complexity of ventricular response in atrial fibrillation. Discrete Dynamics In Nature And Society, 4, 63-75.
