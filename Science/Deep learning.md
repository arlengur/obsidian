1. Neural Networks and Deep learning
2. Improving Deep Neural Networks
3. Structuring your ML project
4. Convolutional Neural Networks (CNN)
5. Natural Language Processing (RNN)

Примеры сетей для решения задач
- Standart NN - продажа домов, реклама
- CNN - изображения
- Recurent NN - аудио, NLP
- Custom/Hybrid NN - автопилот

Структуированные данные - данные в таблице когда каждый параметр (размер дома, имя пользователя, возраст) определен.

Неструктуированные данные - аудио, картинка, текст

Символом m обозначаем размер входных данных

> Логистическая регрессия — это алгоритм бинарной классификации. 

Пример: 
На вход подается некоторое изображение, а на выходе нужно получить метку, сообщающую, есть ли на изображении кошка: если есть, метка равна 1, а если нет — 0. Эту выходную метку мы обозначим за y. 

Обучающий пример $(x, y):x \in R^{n_x}, y \in \{0, 1\}$
Количество обучающих примеров - m: $\{(x^1, y^1), (x^2, y^2), ... (x^m, y^m)\}$
$M=M_{train}$ - число обучающих примеров
$M=M_{test}$ - число тестовых примеров
$X=[x^1, x^2,...x^m]$ - матрица $[n_x \cdot m]$ входных векторов обучающей выборки, где m - число обучающих примеров, $n_x$ - размерность. $X \in R^{n_x \cdot m}$
$Y=[y^1, y^2,...y^m]$ - матрица $[1 \cdot m]$ выходных данных

$\widehat{y}=P(y=1|x)$ - вероятность того что y=1 при входных параметрах х
параметры: $w \in R^{n_x}, b \in R$, 
тогда $\widehat{y}=\sigma(w^Tx+b)$, где $z=w^Tx+b$
$\sigma(z) = {1 \over {1+e^{-z}}}$
Функция потерь: $L(\widehat{y},y)=1/2(\widehat{y}-y)^2$  но на нем плохо работает метод градиентного спуска, поэтому будем пользоваться такой функцией $L(\widehat{y},y)=-(ylog\widehat{y}+(1-y)log(1-\widehat{y}))$

Функция стоимости оценивает качество работы алгоритма на обучающей выборке $J(w, b)={1 \over m} \sum_{i=1}^mL(\widehat{y}^i,y^i)$ то есть эффективность параметров w и b на обучающей выборке. Эта функция выпуклая.

Алгоритм градиентного спуска: от выбранной начальной точки делается шаг в направлении, где уклон вниз круче всего и так далее. 
$w = w - \alpha{dJ(w) \over dw}=w - \alpha dw$
$\alpha$ - скорость обучения. Она задает размер шага, который мы делаем на каждой итерации градиентного спуска. 
$dw={dJ(w) \over dw}$ - производная (приращение, которое мы хотим сообщить параметру w)

> производная — это тангенс угла наклона касательной к функции в данной точке, то есть отношение вертикального и горизонтального катетов прямоугольного треугольника, гипотенуза которого идет по касательной к J(w) в этой точке.

Если мы учитываем значение b, то формула примет вид
$w = w - \alpha{\partial J(w,b) \over \partial w}$
$b = b - \alpha{\partial J(w,b) \over \partial b}$
$\partial b$ - частная производная от функции многих параметров
$db$ -  производная от функции 

# Графическое представление производной

функция f(x)=3x
первая точка x=2: f(x)=6
вторая точка x=2.001: f(x)=6.003
Производная тогда будет равно отношению катетов этого треугольника: ${0.003 \over 0.001} = 3 = f'(x)$
```functionplot
---
bounds: [0,5,0,10]
disableZoom: true
grid: true
---
f(x)=3x
```










