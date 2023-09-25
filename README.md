# Atrial Fibrillation (AFib) binary classifier
這個專案的目標在於實作一個簡易的、能夠自動偵測心房纖維性顫動(Atrial Fibrillation)的二元分類器，好協助居家照護者或病友能夠及早發現心律異常，在有限時間內做出正確的醫療決策。 
# Abstract:

這個專案引用了這篇論文[1]的特徵工程方法，並根據實際情況做出了一定幅度的修改，目的在於增加分類器的穩健性(Robustness)及可解釋性(Interpretability)。

演算流程:
1. 將每一段長達30分鐘的ECG做預處理、透過neurokit內建的peak detection algorithm取得它的心律變異性(Heart rate variability, HRV)。
2. 將HRV畫成Poincare plot可以看見清晰的分群。
3. 使用DBSCAN分群算法計算分群數量(群的尺度大小參考自[2])，另外再計算Poincare plot的統計量，獲得4 dimensions featue values。
4. 透過CatBoostClassfier 對 feature values做ensemble learning。

資料流程架構 :
![alt text](https://github.com/ilovec8763/Atrial-Fibrillation-AFib-binary-classifier/blob/master/Data%20Flow%20Diagram%2C%20DFD.png)

主要函式依賴性 :
![alt text](https://github.com/ilovec8763/Atrial-Fibrillation-AFib-binary-classifier/blob/master/Dependency_of_functions.png)

# Dataset
我在這裡使用PAF Prediction Challenge Database ，這是一個由Computing in Cardiology conference 為了其2001發起的挑戰而構建的database，這個挑戰的目標是開發預測陣發性心房顫動 (PAF) 的自動化方法。

該database分為學習集（名稱格式為 n *和 p *的記錄）和測試集（名稱格式為 t *的記錄）。此database中含有100組30分鐘的含標籤的ECG片段，一半包含AFib訊號(p)，一半是正常訊號(n)。
另外還有不含標籤的測試集紀錄以及AFib發作後5分鐘的ECG訊號(以c做開頭)，這裡我們不會用到。

若想知道更詳細的資訊可至physionet.org 的官網了解 

PAF Prediction Challenge Database: https://www.physionet.org/content/afpdb/1.0.0/

# Result
Let AFib = Positive, Normal = Negative.

Precision : 92%.

Recall : 80%.

F1-score : 85.5%.

Accuracy : 84%.

# Pictures for demonstration

## Dataset distribution
官方提供的數據集中，AF的訊號跟正常訊號各占一半，故不用考慮imbalanced data的問題。
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/data%20balance.png)

## Sanity check for peak detection algorithm
使用neurokit 2 內建的演算法neurokit (default)，基於absolute gradient 的 steepness 偵測QRS complex，算法細節詳見 [https://github.com/neuropsychology/NeuroKit/issues/476](https://joss.theoj.org/papers/10.21105/joss.02621)

![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/n10_%E6%AD%A3%E5%B8%B8%E6%83%85%E5%BD%A2_ECG.png)


## Poincaré plot of a normal person
Poincaré plot 是一種將RR間隔（心臟跳動間的時間間隔）與下一個RR間隔進行散點圖表示的方法。這個圖會呈現出一個雲狀分布，並且會沿著對角線的方向排列。雲的形狀提供了非常有用的心率變異性（HRV）描述訊息。雲的長度(SD1)對應於長期變異性的程度，而寬度(SD2)表示短期變異性[3]。

圖中可看到一個正常人ECG的Poincaré plot，以及用DBSCAN分群上色後的Poincaré plot。

![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/poincare_plot_normal.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/poincare_plot_normal_colored.png)

## Distribution of number of clusters of normal people's Poincare plot
正常人的 Poincaré plot 群數大多較少。

![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/Clusters%20of%20normal.png)

## Distribution of number of clusters of AFib patients' Poincare plot 
 AFib病人的Poincaré plot 群數明顯偏多。
 
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/Cluster%20of%20AFib%20patients.png)

## Poincaré plot of an AFib patient
圖中可看到一個AFib病患ECG的Poincaré plot，以及用DBSCAN分群上色後的Poincaré plot。

![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/poincare_plot_AFib.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/poincare_plot_AFib_colored.png)

## Performance of SVM on feature values 
依照原作論文作者的算法，先用kmeans計算Poincaré plot的群數量，再用dispresion和stepping作為feature，使用kmeans分成上下兩群，將正常與病態ECG做分流之後再做SVM，見下圖。
可以看到黑色方形代表AFib患者的樣本，黑色x代表正常人的樣本，兩個群體在原作者的算法下可以稍微看到模糊的邊界。

![alt text](https://github.com/ilovec8763/Atrial-Fibrillation-AFib-binary-classifier/blob/master/%E5%8E%9F%E8%AB%96%E6%96%87_%E5%9C%96%E7%89%87.png)

但kmeans分群很容易受到雜訊影響，特別是當peak detection 演算法若沒有經過特別挑選，很難重現論文的結果，且Poincaré plot用kmeans分群的計算結果解釋性較弱，整體缺乏魯棒性，對於容易產生artifacts的攜帶裝置而言可能較為不利。

基於特徵工程之後的features若直接使用kernel SVM學習，容易發生overfiting (仿照論文作法，參數用gridsearch 計算最佳參數)，此乃歸因於資料集太小。

![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/SVM%20cm.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/SVM%20learning%20curve.png)

## Performance of CatBoostClassifier on feature values
基於上一章節的討論，為改進實驗結果，使用DBSCAN作為計算Poincaré plot群數量的算法以提升準確度，並且使用tree based algorithm的ensemble learning以提升函數的非線性以及避免overfit。
如此一來，不需要使用kmeans及資料分流，也能達到略差的準確度84%(論文91.4%)及相當接近的特異度90%(論文92.9%)。

![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/CatBoost_Learning_Curve.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/confusion_matrix.png)
![alt text](https://github.com/ilovec8763/Physiological-Signal-Processing-/blob/master/Non_normalized_cm_catbosst.png)



# Reference
[1] Park, J., Lee, S., & Jeon, M. (2009). Atrial fibrillation detection by heart rate variability in Poincare plot. Biomedical engineering online, 8(1), 1-12.

[2] Käsmacher, H., Wiese, S., & Lahl, M. (2000). Monitoring the complexity of ventricular response in atrial fibrillation. Discrete Dynamics In Nature And Society, 4, 63-75.

[3] Khandoker, A. H., Karmakar, C., Brennan, M., Palaniswami, M., & Voss, A. (2013). Poincaré plot methods for heart rate variability analysis. Boston, MA, USA: Springer US.
