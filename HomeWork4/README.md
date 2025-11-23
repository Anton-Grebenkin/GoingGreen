### Архитектурная ката [Going Green](http://nealford.com/katas/kata?id=GoingGreen "Going Green")

#### 1. **Кейс**

За основу взят кейс из [предыдущего домашнего задания](https://github.com/Anton-Grebenkin/GoingGreen/tree/main/HomeWork1) 

#### 2. **Диаграмма развертывания**
Вся инфраструктура размещена в одном ЦОД-е. Сервисы разворачиваются в k8s кластере и общаются между собой по http, кластер СУБД Postgreql разворачивается отдельно. У каждого сервиса отдельная база данных внутри кластера. Взаимодействия с сервисами из вне осуществляются с помощью API Gateway.

![enter image description here](https://github.com/Anton-Grebenkin/GoingGreen/blob/main/HomeWork4/hm4.png)
