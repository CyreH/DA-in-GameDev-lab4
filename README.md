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
Разобраться в работе перцептрона.

## Задание 1
### В проекте Unity реализовать перцептрон, который умеет производить вычисления
#### Ход работы
- Создать новый проект Unity и привязать к пустому объекту скринт перцептрона![image](https://user-images.githubusercontent.com/102403656/205052533-ef6ff704-f8e4-453c-855d-144698a6a6f7.png)



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
