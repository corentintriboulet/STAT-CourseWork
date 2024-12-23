---
title: "Impact du CO2 sur la température de la planète"
output:
  html_document: default
  pdf_document: default
---

Pour de l'aide sur R Markdown on peut aller sur

<http://rmarkdown.rstudio.com>\>

<https://lms.fun-mooc.fr/c4x/UPSUD/42001S02/asset/RMarkdown.html>

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, fig.width = 6, fig.height = 6)
```

## Introduction

On s'intéresse à la dépendance entre la température de surface de la planète et les émissions de C02 et leurs évolutions dans le temps. L'analyse de la température de surface GISS ver. 4 (GISTEMP v4) est une estimation du changement global de la température de surface. Les données proviennent de NOAA GHCN v4 (stations météorologiques) et ERSST v5 (zones océaniques). Plus de détails sont accessibles sur ce lien <https://data.giss.nasa.gov/gistemp/>. Concernant le CO2, il est mesuré sur le Mauna Loa (sommet de l'archipel d'Hawai) depuis la fin des annnées 50. Détails sur <https://gml.noaa.gov/ccgg/trends/>

Le fichier "climat_CO2.txt" contient 3 colonnes :

-   **year** : années entre 1959 et 2020

    **CO2** : CO2 exprimé en fraction molaire dans l'air sec, micromol/mol, abrégé en ppm. Voir www.esrl.noaa.gov/gmd/ccgg/trends/ pour plus de détails.

-   **gistemp** : anomalies de température en dégrés Celsius. Il s'agit des écarts entre la température annuelle et la température moyenne de la période 1951-1980.

## 1 - On commence par lire et visualiser les données du fichier

```{r}
CO2 = read.table(file = "climat_CO2.txt", header = TRUE)
head(CO2)
pairs(CO2)
plot(CO2$year,CO2$CO2)
plot(gistemp~CO2, data = CO2)
```

On remarque que l'objet CO2 est une structure de données. L'accès aux différents champs se fait avec le \$. Mais la table CO2 peut aussi être utilisée comme une matrice. Les commandes pour accéder à une colonne, à une ligne, à un set d'indices sont alors les suivantes :

```{r}
CO2[,3]
CO2[30,]
indices = CO2$gistemp>0.5
CO2[indices,1]
CO2[c(12, 35, 50),]
```

## 2 - On met en place un modèle de régression de la température en fonction du CO2

La commande summary permet d'avoir accès aux estimations des paramètres et aux tests statistiques. La commande plot permet l'analyse des résidus.

```{r}
mod1 = lm(gistemp~CO2, data = CO2)
summary(mod1)
par(mfrow=c(2,2))
plot(mod1)
```

```{r}
plot(gistemp~CO2, data = CO2)
abline(mod1$coefficients, col = 'red')
```

## 3 - Prédiction du modèle

### La prédiction aux points d'observation

Le champ *fitted.values* de l'objet *CO2* contient les valeurs de $\hat{y}_i , \forall i \in \{1, ..., n\}$ .

```{r}
plot(CO2$gistemp ,mod1$fitted.values)
```

On considère que les résidus traduisent un bruit de mesure. Question : quelle est la donnée ayant la plus grande erreur de mesure ?

```{r}
ecarts = abs(CO2$gistemp -mod1$fitted.values)
indice = ecarts == max(ecarts)
CO2[indice,]
c(CO2$gistemp[indice] ,mod1$fitted.values[indice])
```

### La prédiction en dehors des points d'observation

On utilise la fonction *predict* sur un seul point :

```{r}
nouvel_individu = data.frame(CO2 = 380)
prediction = predict(mod1, newdata = nouvel_individu)
plot(gistemp~CO2, data = CO2)
abline(mod1$coefficients, col = 'red')
points(nouvel_individu$CO2,prediction,col ="blue", pch = 19)
```

On peut aussi obtenir les intervalles de confiance et de prédiction :

```{r}
prediction_IC = data.frame(predict(mod1, newdata = nouvel_individu, interval = 'confidence', level = 0.95))
prediction_IC
plot(gistemp~CO2, data = CO2)
abline(mod1$coefficients, col = 'red')
points(nouvel_individu$CO2,prediction_IC$fit,col ="blue", pch = 19)
points(nouvel_individu$CO2,prediction_IC$lwr,col ="blue", pch = 19)
points(nouvel_individu$CO2,prediction_IC$upr,col ="blue", pch = 19)

prediction_IP = data.frame(predict(mod1, newdata = nouvel_individu, interval = 'prediction', level = 0.95))
points(nouvel_individu$CO2,prediction_IP$lwr,col ="blue", pch = 19)
points(nouvel_individu$CO2,prediction_IP$upr,col ="blue", pch = 19)

```

## Question 2

Donner l’équation du modèle

```{r}
mod2 = lm(CO2~year, data = CO2)
summary(mod2)
par(mfrow=c(2,2))
plot(mod2)
```

CO2 = -2826.2821 + 1.5997\* year\
98.3 pourcent

```{r}
mod2_poly = lm(CO2 ~ poly(year, 2), data = CO2)  
summary(mod2_poly)

par(mfrow=c(2,2))
plot(mod2_poly)

par(mfrow=c(1,1))
plot(CO2 ~ year, data = CO2)
lines(CO2$year, fitted(mod2_poly), col = 'red')
```

## Question 3

```{r}
nouvel_individu2 = data.frame(year = 2050)
prediction2_IP=data.frame(predict(mod2_poly, newdata = nouvel_individu2, interval = 'prediction', level = 0.95))
prediction2_IP

```

```{r}
nouvel_individu3 = data.frame(CO2 = prediction2_IP$lwr)
prediction3_IP=data.frame(predict(mod1, newdata = nouvel_individu3, interval = 'prediction', level = 0.95))
prediction3_IP

nouvel_individu4 = data.frame(CO2 = prediction2_IP$upr)
prediction4_IP=data.frame(predict(mod1, newdata = nouvel_individu4, interval = 'prediction', level = 0.95))
prediction4_IP
```

```{r}
pessimiste=prediction4_IP$upr
pessimiste
optimiste=prediction3_IP$lwr
optimiste
```
