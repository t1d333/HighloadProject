# Проект по курсу "Проектирование высоконагруженных систем".

## 1. Тема и целевая аудитория

**Twitch** – это стриминговый сервис, предназначенный для трансляции потоковых видео на игровую тематику. Сервис специализируется на играх — пользователи могут наблюдать за геймплеем, киберспортивными турнирами. Смотреть можно как онлайн, так и в записи.

### Целевая аудитория

По состоянию на август 2023 года Twitch посещает 140 миллионов уникальных пользователей каждый месяц [ \[ 1 \] ][2].

[Распределение аудитории][3]

| Страна       | Количество пользователей, млн | процент от общего количества |
| ------------ | ----------------------------- | ---------------------------- |
| США и Канада | 93                            | 36.32%                       |
| Бразилия     | 16.9                          | 6.6%                         |
| Германия     | 16.8                          | 6.56%                        |
| Англия       | 13.4                          | 5.23%                        |
| Франция      | 11.3                          | 4.41%                        |
| Россия       | 10.5                          | 4.1%                         |
| Испания      | 10.5                          | 4.1%                         |
| Аргентина    | 10                            | 3.9%                         |
| Мексика      | 9.2                           | 3.59%                        |
| Италия       | 8.3                           | 3.24%                        |
| Турция       | 7.5                           | 2.92%                        |
| Южная Корея  | 6.7                           | 2.61%                        |
| Польша       | 4.8                           | 1.87%                        |
| Япония       | 4.1                           | 1.6%                         |
| Австралия    | 4.1                           | 1.6%                         |

Большая часть аудитории приходится на Северную Америку — 39.91%. На втором месте Европа — 32.43%. На третьем месте Южная Америка — 10.5%.

### MVP

- Регистрация пользователей
- Проведение/просмотр прямых трансляций
- Просмотр закончившихся трансляций
- Подписки на пользователей
- Поиск трансляций
- Чат трансляции

## 2. Расчет нагрузки

### 2.1 Продуктовые метрики

- Месячная аудитория(MAU) — 140 млн пользователей
- Дневная аудитория(DAU) — 31 млн пользователей

Для пользователя храним следующую информацию:

- Аватар
- Никнейм
- Баннер канала
- Описание канала

Для экономии места можно конвертировать изображения в WebP.
Итого средний размер хранилища пользователя $\approx 4 MB$

#### Среднее количество действий пользователя по типам

- Средняя суммарная длительность всех трансляций за один день — 2 225 167 часов
- Среднее число трансляций в день — 1 167 398

Тогда средняя продолжительность трянсляции: $2225167 \div 1167398 \approx 1.9$

По [статистике][4] пользователь тратит в день суммарно 95 минут на просмотр прямых трансляций на Twitch.
Можно предположить, что большинство пользователей не тратят все время на одну трансляцию, тогда в среднем пользовель посещает $\approx$ 3-4 трансляции в день и тратит на каждую около 30 минут.

На основе наблюдения за [количеством сообщений в секунду][5], можно сказать, что среднее количество примерно 600 сообщений. Тогда в день — $600 \times 60 \times 60 \times 24 \approx 52$ млн сообщений в день.

Таблица действий для одного пользователя:

| Тип действия             | частота |
| :----------------------- | :------ |
| Просмотр трянсляции      | 3       |
| Подписка на пользователя |         |
| Поиск трансляции         |         |

Таблица действий, которые нельзя рассчитать однозначно для одного пользователя:

| Тип действия             | частота\[млн/день\] |
| :----------------------- | :------------------ |
| Проведение трансляции    | 1.1                 |
| Отправка сообщения в чат | 52                  |

### 2.2 Технические метрики

#### Размер хранения в разбивке по типам данных

##### Хранилище записей трансляций

На платформе Twitch срок хранения трансляции зависит от статуса пользователя.

| Категория                                   | срок хранения |
| :------------------------------------------ | :------------ |
| Партнеры, пользователи Twitch Prime и Turbo | 60 дней       |
| Компаньоны                                  | 14 дней       |
| Остальные                                   | 7 дней        |

По [статистике](https://streamscharts.com/overview/partners) общее количество партнеров Twitch составляет **63 тыс**, а общее число компаньонов — **2.1 млн**. Допустим, что все пользователя сохраняют свои трансляции.
Допутстим, каждый партнер проводит 5 трансляций в неделю (в месяц примерно 24 трансляции), со средней продолжительностью 3,5 часа, с характеристиками трансляции 1080 60fps, тогда на хранение трансляций партнеров за 2 месяца требуется:

$$
6000 \times 3.5 \times 60 \times 60 \times 63000 \times 48 \approx 26614 \ \text{ГБайт}
$$

Для компаньонов возьмем те же характеристики, тогда на хранение трансляций компаньонов за 14 дней необходимо:

$$
6000 \times 3.5 \times 60 \times 60 \times 21 \times 10^{6} \times 10 \approx 184820 \ \text{ГБайт}
$$

Для всех остальных(их примерно [14.5 млн](https://streamscharts.com/overview/partners) — в статистике учитывались только каналы с среднем количеством зрителей 5 и более) возьмем следующие характеристики:

- Качество — 720p
- Частота кадров — 60fps
- Продолжительность трансляции $\approx$ 3.5 часа
- Количество трансляций в неделю — 4
  Итого: 382843 ГБайт необходимо на хранение трансляций пользователй

$$
4500 \times 3.5 \times 60 \times 60 \times 4 \times 14.5 \times 10^{6} \approx 382843 \ \text{ГБайт}
$$

##### Промежуточное хранилище

##### Хранилище сообщений

#### Сетевой трафик

##### Входящий трафик

Как уже указывалось выше, средняя суммарная длительность всех трансляций за один день — 2 225 167 часов. Для видео в качестве 720p и частотой обновления обновления 60 fps, платформа Twitch [рекомендует][6] выставлять битрейт 4500 kbps.
Поэтому входящая нагрузка составляет:

$$
4500 \times2\ 225\ 168  \times 60 \times 60 \approx 3.6 \times 10^{13} \ \text{кбит в сутки} \approx 4 \ 250 \ 000 \ \text{ГБайт в сутки}
$$

Из [данных][1] следует, что самое большое количество одновременно активных трансляций в течении дня составляет примерно 130 000, тогда пиковая нагрузка составляет:

$$
130\ 000 \times 4500 \approx 557 ~ \text{Гбит в секунду}
$$

Суточное количество сообщений в чат составляет $\approx$ 52 млн сообщений.
Допустим, символы кодируются в UTF-8, тогда на каждый символ приходится от 1 до 4 байт, возьмем средний размер — 2 байта. Пусть средняя длина сообщения составляет 15 символов, тогда дневная нагрузка рассчитывается следующим образом:

$$
52 \times 10^{6}\times 16 \times 15 \approx 12.5 ~ \text{Гбит в день}
$$

| Тип запроса              | суточная нагрузка, ГБайт/сутки | пиковая нагрузка, Гбит/сек |
| :----------------------- | :----------------------------- | :------------------------- |
| Проведение трансляции    | 4 250 000                      | 557                        |
| Просмотр транляции       |                                |                            |
| Отправка сообщений в чат |                                |                            |

##### Исходящий трафик

#### RPS в разбивке по типам запросов

| Тип запроса | RPS |
| :---------- | :-- |
|             |     |

## Источники

1. https://twitchtracker.com/statistics
2. https://www.demandsage.com/twitch-users/
3. https://worldpopulationreview.com/country-rankings/twitch-users-by-country
4. https://www.enterpriseappstoday.com/stats/twitch-statistics.html
5. https://stats.streamelements.com/
6. https://help.twitch.tv/s/article/broadcasting-guidelines?language=en_US
7. https://twitchstats.net/
8. https://streamscharts.com/

[1]: https://twitchtracker.com/statistics
[2]: https://www.demandsage.com/twitch-users/
[3]: https://worldpopulationreview.com/country-rankings/twitch-users-by-country
[4]: https://www.enterpriseappstoday.com/stats/twitch-statistics.html
[5]: https://stats.streamelements.com/
[6]: https://help.twitch.tv/s/article/broadcasting-guidelines?language=en_US
[7]: https://twitchstats.net/
[8]: https://streamscharts.com/
