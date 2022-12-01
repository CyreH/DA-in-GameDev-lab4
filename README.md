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
- Для каждой логической функции заполнить список элементов нужными значениями
1. OR: проведя несколько тестовых запусков выяснил, что 4 этапов всегда достаточно для обучения![image](https://user-images.githubusercontent.com/102403656/205056962-66cd527e-c1b5-4fa7-9b5d-a976f81eb701.png)
2. AND: по итогам тестов, максимальное количество этапо для обучение выявилось 9, хотя зачастую всё проходит и за 5-7 этапов![image](https://user-images.githubusercontent.com/102403656/205060966-88aff38a-79ff-4da0-a69c-1161a70da5f4.png)
3. nAND: так же, как и для AND потребовалось максимум 9 этапов![image](https://user-images.githubusercontent.com/102403656/205062289-8ad6ed47-afdb-4f27-a7e5-4ddf6c011701.png)
4. XoR: у перцептрона не получилось обучиться даже за 10 000 этапов![image](https://user-images.githubusercontent.com/102403656/205063158-b42976fc-e975-4e1d-b55b-44a75e99c77c.png)




## Задание 2
### Построить график зависимости количества эпох от ошибки обучения. Указать от чего зависит необходимое кол-во эпох
- ![image](https://user-images.githubusercontent.com/102403656/205072019-dd9efb69-a9f2-4e5e-8275-a3b30a660309.png)
- Количество эпох зависит нескольких факторов:
- - от weights(вес), значение которого мы получаем при успешном выполнении заданного условия и порогового значения, пройти который можно набрав определёнынй "вес"
- - от логической фун


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
