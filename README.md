## A/B test

A/B тестирование - метод исследования для оценки эффективности двух вариантов одного элемента. Классичекое проведение A/B теста проходит следующим образом:

  
  1. **Сформулировать гипотезую.** Гипотеза должна быть конкретной, измеримой и достижимой. Например, гипотеза может заключаться в том, что изменение цвета кнопки с призывом к действию увеличит количество кликов.
  
  
  2. **Определить переменные и вариации.**  Например, вариантами могут быть различные цвета кнопки.
  
 
  3. **Разделить трафик и собрать данные.** После создания вариант, нужно равномерно распределить трафик пользователей между ними. После разделения трафика  необходимо собрать данные о том, как работает каждый вариант.
  
 
  4. **Проанализировать данные.**  Проанализировать данные можно с помощью инструментов статистического анализа. Можете использовать t-критерий, критерий хи-квадрат или z-критерий для определения статистической значимости.
  
 
  5. **Сделать выводы.** На основе анализа делаются выводы о том, какой вариант показал лучшие результаты и почему, чтобы использовать их для улучшения веб-сайта или маркетинговой кампании.

  Данные были вязты с сайта kaggle. Набор данных состоит из 4 полей:
  * user_id - уникальный идентификатор каждого пользователя
  * group - версия старинцы. Исходная версия (control) и новая версия (test)
  * views - количество просмотров страницы
  * clicks - количество 

## Решение 

1. Загружаем данные
```
df = pd.read_csv('ab_test_results_aggregated_views_clicks_2.csv')
```

2. **Сформулируем гипотезу:** на новой версии CTR выше, значит реклама работает эффективнее. При меньшем количестве затрат сможем получить больше продаж.
3. Расчитаем метрику CTR - показатель кликабельности. Чтобы рассчитать показатель, нужно знать количество показов и кликов

CTR = количество кликов / количество показов × 100%
```
df['ctr'] = round(df['clicks'] / df['views'], 3)
```
4. Разделим CTR между двумя версиями страницы, т.е. исходной версией (control) и новой версией (test)
```
test = df[df['group'] == 'test']

control = df[df['group'] == 'control']
```

Обычно рекомендуется разделить вариантную и контрольную группы поровну при A/B тестировании, чтобы гарантировать, что любые различия в результатах теста не вызваны неравномерным распределением участников между группами.

В нашем случае группы разделены поровну.

5. Визуализируем значение CTR для контрольной и тестовой группы
```
import matplotlib.pyplot as plt

val = [test['ctr'].sum(), control['ctr'].sum()]
label = ['Тестовая группа', 'Контрольная группа']
plt.bar(label, val)
plt.title('Сравнение CTR')
plt.show()
```
![alt text](https://github.com/Elisavile/a-b-test/blob/main/hist.png)

Из графика можно сделать вывод, что тестовая группа имеет более высокий CTR

6. Проведем статистические тесты

**Z тест**

* H0: нет существенной разницы между контрольной и тестовой группой (CTR равны)

* H1: существенная разница между контрольной и тестовой группой есть (CTR не равны)

* alpha = 0.05
```
import scipy.stats as stats
sample_size = 60000
alpha = 0.05


z_score = (test.mean()-control.mean())/(control.std()/np.sqrt(sample_size))
print('Z значение :',z_score)


z_critical = stats.norm.ppf(1-alpha)
print('Z критическое :',z_critical)


if z_score >  z_critical:
    print("Нулевая гипотеза отклоняется")
else:
    print("Нулевая гипотеза принимается")
```
Z значение : 8.53332483047081
Z критическое : 1.6448536269514722
Нулевая гипотеза отклоняется

Можно сделать вывод, что есть статистически значимая разница между контрольной и вариационной группами.

**Тест Манна-Уитни**

* H0: нет существенной разницы между контрольной и тестовой группой (CTR равны)
* H1: существенная разница между контрольной и тестовой группой есть (CTR не равны)
* alpha = 0.05
```
stats.mannwhitneyu (test['ctr'], control['ctr'], alternative='two-sided')
```
MannwhitneyuResult(statistic=1826996554.5, pvalue=8.259463834737971e-13)

pvalue < 0.05, значит мы отвергаем нулевую гипотезу в пользу альтернативной. Статистически значимая разница между группами есть

## Вывод
Новая версия рекламы действительно эффективнее, так как CTR выше. Использование изменений в тестовой группе имеет большую кликабельность и вовлеченность пользователей, что может привести к большему количеству продаж при меньших затратах на рекламу. В данном случае, ваша первоначальная гипотеза оказывается подтверждена на основе проведенного z-теста и теста Манна-Уитни.
