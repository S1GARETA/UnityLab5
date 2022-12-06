# Лабораторная работа № 5 Создание индивидуальной системы достижения пользователя и ее интеграция в пользовательский интерфейс
Отчет по лабораторной работе #5 выполнил(а):
- Данькин Сергей Викторович
- РИ-300016

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[Проект Dragon Picker](https://github.com/S1GARETA/Dragon_Picker)

[Шаблон на Яндекс Игры](https://yandex.ru/games/app/197772?draft=true&lang=ru)

## Цель работы

#### Cоздание интерактивного приложения с рейтинговой системой пользователя и интеграция игровых сервисов в готовое приложение.

## Задание 1

#### Используя видео-материалы практических работ 1-5 повторить реализацию приведенного ниже функционала:

1. Интеграции авторизации с помощью Яндекс SDK.
2. Сохранение данных пользователя на платформе Яндекс Игры.
3. Сбор данных об игроке и вывод их в интерфейсе.
4. Интеграция таблицы лидеров.
5. Интеграция системы достижений в проект.

## Ход работы

##### 1. Интеграции авторизации с помощью Яндекс SDK.

Создаём на сцене объект YG, чтобы SDK заработал. Также создаем пустой объект YandexManager, в котором будут храниться все скрипты для яндекса.
Создаем скрипт [CheckConnectYG](https://github.com/S1GARETA/Dragon_Picker/blob/main/DragonPicker/Assets/_Scripts/CheckConnectYG.cs), в котором будет осуществляться проверка подключения SDK и авторизация пользователя.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using YG;

public class CheckConnectYG : MonoBehaviour
{
    private void OnEnable() => YandexGame.GetDataEvent += CheckSDK;
    private void OnDisable() => YandexGame.GetDataEvent -= CheckSDK;
    void Start()
    {
        if (YandexGame.SDKEnabled == true)
        {
            CheckSDK();
        }
    }

    public void CheckSDK()
    {
        if (YandexGame.auth == true)
        {
            Debug.Log("User authorization ok");
        }
        else
        {
            Debug.Log("User not authorization");
            YandexGame.AuthDialog();
        }
    }
}
```

Создадим билд проекта и загрузим его на Яндекс Игры, чтобы проверить, работает ли код.
Если пользователь авторизаван, то он спокойно может начать играть. В консоле появится соответствующая надпись.

![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.1.jpg)

Иначе, пользователю предложат авторизоваться.

![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.2.jpg)

##### 2. Сохранение данных пользователя на платформе Яндекс Игры.

Для сохранения данных пользователя, нужно создать собственную переменную в скрипте [SavesYG](https://github.com/S1GARETA/Dragon_Picker/blob/main/DragonPicker/Assets/YandexGame/WorkingData/SavesYG.cs).

```cs
namespace YG
{
    [System.Serializable]
    public class SavesYG
    {
        public bool isFirstSession = true;
        public string language = "ru";
        public bool feedbackDone;
        public bool promptDone;

        // Ваши сохранения
        public int score;
    }
}
```

Также нужно дополнить скрипт [DragonPicker](https://github.com/S1GARETA/Dragon_Picker/blob/main/DragonPicker/Assets/_Scripts/DragonPicker.cs), где будет производиться сохранение прогресса и вывод количества очков в консоль при старте игры и проигрыше.
Примечание: тут показаны только добавленные строчки.

```cs
public class DragonPicker : MonoBehaviour
{
    private void OnEnable() => YandexGame.GetDataEvent += GetLoadSave;
    private void OnDisable() => YandexGame.GetDataEvent -= GetLoadSave;

    [SerializeField] private TextMeshProUGUI scoreGT;

    void Start()
    {
        if(YandexGame.SDKEnabled == true)
        {
            GetLoadSave();
        }
    }

    public void DragonEggDestroyed()
    {
        if (shieldList.Count == 0)
        {
            GameObject scoreGO = GameObject.Find("Score");
            scoreGT = scoreGO.GetComponent<TextMeshProUGUI>();
            UserSave(int.Parse(scoreGT.text));

            GetLoadSave();
        }
    }

    public void GetLoadSave()
    {
        Debug.Log(YandexGame.savesData.score);
    }

    public void UserSave(int currentScore)
    {
        YandexGame.savesData.score = currentScore;
        YandexGame.SaveProgress();
    }
}
```

Создаем билд и проверяем работоспособность на Яндекс Игры.

При старте игры, в консоле показывается то количество очков, которое у нас было.

![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.3.jpg)

Наберём какое-нибудь количество очков и проиграем. После проигрыша набранные очки пишутся в консоль.

![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.4.jpg)

Перезайдем в игру и посмотрим, сохранились ли наши очки с прошлой попытки. Сверху очки, которые мы набрали после проигрыша. Снизу, те очки, которые были набраны при прошлой попытке. Они совпадают.

![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.5.jpg)

##### 3. Сбор данных об игроке и вывод их в интерфейсе.

Создаём в главном меню canvas с лучшими очками.

![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.6.jpg)

А на игровой сцене над магом будем выводить имя игрока.

![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.7.jpg)

Дописываем в скриптах SavesYG, DragonOicker и ChechConnectYG нужные строки, чтобы данные лучшего количества очков записывались для каждого пользователя по отдельности.

##### [SavesYG](https://github.com/S1GARETA/Dragon_Picker/blob/main/DragonPicker/Assets/YandexGame/WorkingData/SavesYG.cs)
```cs
namespace YG
{
    [System.Serializable]
    public class SavesYG
    {
        public bool isFirstSession = true;
        public string language = "ru";
        public bool feedbackDone;
        public bool promptDone;

        // Ваши сохранения
        public int score;
        public int bestScore = 0;
    }
}
```
##### [DragonPicker](https://github.com/S1GARETA/Dragon_Picker/blob/main/DragonPicker/Assets/_Scripts/DragonPicker.cs)
```cs
    public void GetLoadSave()
    {
        Debug.Log(YandexGame.savesData.score);
        GameObject playerNamePrefabGUI = GameObject.Find("PlayerName");
        playerName = playerNamePrefabGUI.GetComponent<TextMeshProUGUI>();
        playerName.text = YandexGame.playerName;
    }

    public void UserSave(int currentScore, int currentBestScore)
    {
        YandexGame.savesData.score = currentScore;
        if (currentScore > currentBestScore)
        {
            YandexGame.savesData.bestScore = currentScore;
        }
        YandexGame.SaveProgress();
    }
```
##### [ChechConnectYG](https://github.com/S1GARETA/Dragon_Picker/blob/main/DragonPicker/Assets/_Scripts/CheckConnectYG.cs)
```cs
    public void CheckSDK()
    {
        if (YandexGame.auth == true)
        {
            Debug.Log("User authorization ok");
        }
        else
        {
            Debug.Log("User not authorization");
            YandexGame.AuthDialog();
        }
        GameObject scoreBO = GameObject.Find("BestScore");
        scoreBest = scoreBO.GetComponent<TextMeshProUGUI>();
        scoreBest.text = "Best Score: " + YandexGame.savesData.bestScore.ToString();
    }
```

В самой игре всё работает.

![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.8.jpg)
![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.9.jpg)

##### 4. Интеграция таблицы лидеров.

Для добавления таблицы лидеров, в скрипте DragonPicker дописываем нужную строчку кода.

```cs
    public void DragonEggDestroyed()
    {
        if (shieldList.Count == 0)
        {
            YandexGame.NewLeaderboardScores("TOPPlayeScore", int.Parse(scoreGT.text));
        }
    }
```
Создаем билд и проверяем.

![img](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.10.jpg)

##### 5. Интеграция системы достижений в проект.

Для создания системы достижений, нужно добавить переменную в скрипте [SavesYG](https://github.com/S1GARETA/Dragon_Picker/blob/main/DragonPicker/Assets/YandexGame/WorkingData/SavesYG.cs).

```cs
public string[] achivMent = new string[5];
```

После в скрипте [ChechConnectYG](https://github.com/S1GARETA/Dragon_Picker/blob/main/DragonPicker/Assets/_Scripts/CheckConnectYG.cs) прописываем строчки кода, которые будут отвечать за проверку наших достижений, и если достижения имеются, то выводить их в специальном окне.

```cs
public void CheckSDK()
    {
        if ((YandexGame.savesData.achivMent)[0] == null & !GameObject.Find("ListAchiv"))
        {

        }
        else
        {
            foreach (string value in YandexGame.savesData.achivMent)
            {
                Debug.Log(value + " Cool!!!");
                GameObject listAchiv = GameObject.Find("ListAchiv");
                achivmentList.text = value.ToString() + "\n";
            }
        }
    }
```

Дальше нужно создать само достижение. Например, если сломаются все щиты, то есть игрок проиграет, то он получит за это достижение.
Для этого, в скрипте [DragonPicker](https://github.com/S1GARETA/Dragon_Picker/blob/main/DragonPicker/Assets/_Scripts/DragonPicker.cs) прописываем код, который будет добавлять достижение, если количество щитов упадёт до нуля.

```cs
public void DragonEggDestroyed()
    {
        if (shieldList.Count == 0)
        {
            string[] achivList;
            achivList = YandexGame.savesData.achivMent;
            achivList[0] = "Береги щиты!";
        }
    }
```

Запускаем и проверяем.

![gif](https://github.com/S1GARETA/UnityLab5/blob/main/Demo%20files/1.1.gif)

## Задание 2

#### Описать не менее трех дополнительных функций Яндекс SDK, которые могут быть интегрированы в игру.

### Ход работы:

1. Баннерная реклама. Её достаточно просто интегрировать и она эффективна для получения дохода.
2. Внутриигровые покупки. Можно предоставить пользователю возможность совершать покупки внутри игры за реальные деньги. Такими покупками могут быть: скины для щита или чего-то другого, дополнительная жизнь и тд.
3. Оценка игры. Можно дать возможность оценить игру. Выставить ей оценку или написать отзыв о ней.

## Выводы

В данной лаборатной работе был изучен важный функционал Яндекс SDK:

1. Авторизация пользователя.
2. Сохранение данных пользователя в облаке.
3. Интеграция таблицы лидеров.
4. Интеграция системы достижений.

Всё что было изучено, стало для меня новой полезной информацией. Возникли некоторые трудности при создании системы достижений, так как код из видео-лекции не работал. Так что пришлось немного самому подумал и решить как-то эту проблему.
