# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #4 выполнил:
- Гусамов Артур Филаритович
- РИ210950
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.

## Задание 1
### Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity.
- Создать новый 3D проект в Unity![image](https://user-images.githubusercontent.com/102403656/198213588-40d16a9d-a47a-4134-8a36-9b7af6e7696e.png)
- Скачать MLAgent и добавить в созданный проект![image](https://user-images.githubusercontent.com/102403656/198215381-764f7847-9b82-42f3-8186-83cd93e7794e.png)
- Пишем серию команд для создания и активации нового ML-агента, а также для скачивания необходимых библиотек:
    - mlagents 0.28.0
    - torch 1.7.1
 ![image](https://user-images.githubusercontent.com/102403656/198271796-33d1ae0f-48a9-45b9-9446-1d1c558e50e6.png)
 ![image](https://user-images.githubusercontent.com/102403656/198271988-30f28357-3de8-42f7-b6e5-0d08463dcb1a.png)
 ![image](https://user-images.githubusercontent.com/102403656/198277240-a8e65778-29c6-483a-9085-20f1f9b4ba32.png)
- Cоздать на сцене плоскость, куб и сферу![image](https://user-images.githubusercontent.com/102403656/198282407-caf0d120-a4a3-4bcc-84df-4ef8013c0729.png)
- Создать скрипт и написать нужный код![image](https://user-images.githubusercontent.com/102403656/198283834-94f24aa9-f0a8-4316-a52e-8d8382c36f53.png)

```c#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}


```
- К объекту "сфера" добавить компоненты Rigidbody, Decision Requester, Behavior Parameters и настроить их![image](https://user-images.githubusercontent.com/102403656/198285102-609efee2-2f1d-4c57-bf3b-3ec75a99086a.png)
- Добавить в корень проекта файл конфигурации нейронной сети, rollerball_config.yaml
```yaml

behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000

```

- Запустить работу MlAgent![image](https://user-images.githubusercontent.com/102403656/198288923-553c87f3-a8bc-4e80-9f21-3145cd967cf4.png)
- Результаты первого запуска![image](https://user-images.githubusercontent.com/102403656/198289457-a9e4a452-90d8-48fc-83da-d5776017baff.png)![image](https://user-images.githubusercontent.com/102403656/198290724-ca3e1746-225a-4fab-a02f-8d1d0b1691eb.png)
- Результат обучения с 3 копиями модели![image](https://user-images.githubusercontent.com/102403656/198292507-01999475-2e18-4ab4-b94e-f51cb6d1709d.png)
- Результат обучения с 9 копиями модели![image](https://user-images.githubusercontent.com/102403656/198292562-894bbfea-9baf-49e9-9c47-e56763e91822.png)
- Результат обучения с 27 копиями модели![image](https://user-images.githubusercontent.com/102403656/198293199-d5a9486a-528d-456b-b1ee-f5c3c5eafad9.png)
- Далее обученую модель нужно добавить в ассеты![image](https://user-images.githubusercontent.com/102403656/198294192-4f719686-115d-4e99-8c74-68b34eb00f11.png)![image](https://user-images.githubusercontent.com/102403656/198294258-7018e058-5d77-4f24-86ae-e8bdfbbcf50d.png)
- Добваить обученую модель поведения к поведению сферы

    ![image](https://user-images.githubusercontent.com/102403656/198294604-bc959ccd-bc7b-4297-a202-f991f13706b8.png)
- Резултат поведения модели

    ![Result](https://user-images.githubusercontent.com/102403656/198296390-fdad3375-e6f4-4094-8511-fc3f07fde150.gif)

**Подитог**: по результатам обучения можно сделать вывод, что при больших тестируемых моделях скорость обучения увеливичается


## Задание 2
### Подробно опишите каждую строку файла конфигурации нейронной сети. Найти информацию о компонентах Decision Requester, Behavior Parameters.
```yaml

behaviors:
  RollerBall:                           #Имя агента
    trainer_type: ppo                   #Выбираем тип обучения(Proximal Policy Optimization)
    hyperparameters:                    #Ниже будут задаваться гиперпараметры
      batch_size: 10                    #Количество опытов на каждой итерации для обновления экстремумов функции
      buffer_size: 100                  #Количество необходимого опыта для обновления модели
      learning_rate: 3.0e-4             #Шаг обучения
      beta: 5.0e-4                      #Отвечает за случайные действия
      epsilon: 0.2                      #Порог расхождений между старой и новой политиками при обновлении
      lambd: 0.99                       #Определяет авторитетность оценок значений во времени
      num_epoch: 3                      #Количество проходов через буфер опыта, при выполнении оптимизации
      learning_rate_schedule: linear    #Определяет как скорость обучения изменяется с течением времени
    network_settings:                   #Сетевые настройки
      normalize: false                  #Отключаем нормализацию входных данных
      hidden_units: 128                 #Количество нейронов в скрытых слоях сети
      num_layers: 2                     #Количество скрытых слоёв в сети
    reward_signals:                     #Сигналы о вознаграждении
      extrinsic:                        #
        gamma: 0.99                     #Коэффициент скидки для будущих вознаграждений
        strength: 1.0                   #Коэффициент на который умножается вознаграждение
    max_steps: 500000                   #Общее количество шагов, которые должны быть выполнены в среде до завершения обучения
    time_horizon: 64                    #Количество циклов ML агента, хранящихся в буфере до ввода в модель
    summary_freq: 10000                 #Количество опыта, который необходимо собрать перед созданием и отображением статистики

```
- Decision Requester - запрашивает решение через регулярные промежутки времени и обрабатывает чередование между ними во время обучения
- Behavior Parameters - определяет принятие объектом решений, в него указывается какой тип поведения будет использоваться: уже обученная модель или удалённый процесс обучения


## Задание 3
### Доработать сцену и обучить ML-Agent таким образом, чтобы шар перемещался между двумя кубами разного цвета.
- Добавть ещё один куп в проект
- Доработать скрипт для движения

```c#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public Transform Target1;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
        Target1.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(Target1.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);
        float distanceToTarget1 = Vector3.Distance(this.transform.localPosition, Target1.localPosition);

        if(distanceToTarget < 1.42f || distanceToTarget1 < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}

```
- Переобучить MlAgent![image](https://user-images.githubusercontent.com/102403656/198347668-be55f96d-81fe-409c-a787-0dbdd4854007.png)![image](https://user-images.githubusercontent.com/102403656/198347692-eba98971-b92c-4252-b26e-3ec87745d254.png)

Результат:

![Result1](https://user-images.githubusercontent.com/102403656/198347772-4753ed18-eee9-4a47-916a-f14c5b477b3d.gif)



## Выводы

Игровой баланс - это обязательная часть создания любой качественной игры. Он может включать в себя огромное множества механик, от торговли до баланса оружия. Машинное обучение используется во многих сферах игрового баланса, особенно, когда нужно спроецировать модель из реальной жизни.
Например, для балансирования цен на аукционах часто используются модели, обученные машинным обучением, чтобы у игроков не получилось сломать аукцион своими действиями, тем самым поломав экономику в игре.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
