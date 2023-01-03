# Проект по дисциплине "Языки и методы программирования"
Покупка коттеджей с учётом заданных параметров

# Описание проекта
Данный проект подходит людям, которые желают приобрести коттедж, соответствующий необходимым параметрам. Кроме того, простой и удобный интерфейс позволит любому человеку без проблем подобрать себе варианты коттеджей.

### ***Правила работы с приложением***

# Список задач
- Сгенерировать данные по коттеджам с помощью бибилиотеки Faker для дальнейшей работы с ними
- Реализовать функционал нашего приложения
- Реализовать интерфейс нашего приложения

# Команда
- [Арина Вычегжанина](https://github.com/ArinaVychegzhanina)
- [Мария Воронина](https://github.com/MariVoronina)

# Генерация данных по коттеджам
Генерация была осуществлена нами с помощью библиотеки **Faker**. К сожалению, провайдеры, представленные данной библиотекой, не подходили для наших данных, поэтому нам пришлось писать собственный.

### ***Создание собственного провайдера***
Для начала мы создали класс **Cottage**, который имеет 11 обязательных параметров - 11 характеристик нашего коттеджа: ID коттеджа, цена, год постройки, количество этажей, количество комнат, площадь, наличие электричества, вид отопления, наличие канализации, наличие водоснабжения и адрес.

Затем с помощью модуля **faker.providers** и принадлежащего ему абстрактного класса **BaseProvider** мы написали наш провайдер - класс **CottageProvider**, который наследует класс **BaseProvider** и реализует функцию **create_fake_cottage**. Данная функция и создает данные для одного коттеджа: она ставит в соответствие каждой из 11 характеристик класса **Cottage** некое рандомное значение из указанного нами промежутка или из списка возможных значений.

### ***Запись данных в таблицу***
Мы создаем объект для работы с фековыми данными - **my_faker** и пустой список **data**, куда будем добавлять строки нашей будущей таблицы.
Затем с помощью метода **add_provider** мы для нашего объекта добавляем созданный нами провайдер. И затем собственно создаем строки нашей таблицы: используя цикл **for**, который выполнится 100000 (такое большое число данных необходимо для того, чтобы коттеджи, подходящие по заданным пользователем параметрам обязательно нашлись), на каждой итерации которого мы создаем один фейковый коттедж с помощью функции **create_fake_cottage** и заносим его в словарь, где ключи - это будущие столбцы нашей таблицы, а каждый словарь мы добавляем в наш список, чтобы в итоге у нас получился список словарей - строк нашей таблицы.
В конце мы с помощью модуля **pandas** и его метода **DataFrame** создаем таблицу с данными наших коттеджей, передав методу в аргументы созданный нами списко словарей.

# Реализация функционала
Для работы нашего приложения мы должны реализовать три функции:

- функция, которая в таблице данных по коттеджам находит те из них, которые соответствуют параметрам, указанным пользователем
- функция, которая возвращает список из строк, представляющих собой "красивый вид" наших выбранных строк
- функция, которая по формуле Джеоффриона ищет оптимальный вариант из подобранных ранее вариантов

### ***Реализация функции find_variants***
Данная функция весьма проста в написании. Мы решили, что пользователь будет выбирать не по всем 11 параметрам, а только по 6 самым важным: площади, цене, адресу, наличии электричества, виду отопления и наличии водоснабжения - собственно эти параметры и являются аргументами данной функции.
Так как даты и цены заданы в специальном виде, то мы сначала переводим их в числовой вид для дальнейшего сравнения.
Следующим шагом мы запускаем цикл **for** по строкам нашей таблицы с созданными данными по коттеджам с помощью метода **iterrows**, принадлежащего модулю **pandas**.
На каждой итерации мы проверяем подходят ли данные строки (т.е. параметры коттеджа) под заданные пользователем: если нет - то строка пропускается и происходит следующая итерация, если да - то строка добавляется в новую таблицу **var**, которая специально для этого ранее была создана пустой.
В итоге работы цикла у нас получается таблица с вариантами коттеджей, которые полностью удовлетворяют заданным пользователем параметрам. В качестве результата данная функция возвращает созданную таблицу.

### ***Реализация функции info***
В качестве аргумента на вход в эту функцию подается таблица (в нашем случае это будет таблица, получившаяся в результате выполнения либо функции **find_variants**, либо функции **optimal_dg**). В самой функции создается пустой список **li**, куда далее будут записаны строки, представляющие собой "красивый вывод" данных по коттеджам заданной таблицы (данный список строк необходим для последующего вывода в окне при создании интерфейса).
Далее просто запускаем цикл **for** по строкам заданной таблицы с помощью метода **iterrows**, на каждом шаге которого создаем строку "красивого вывода", используя данные коттеджа, и добавляем ее в наш список. В качестве результата возвращаем созданный список.

### ***Реализация функции optimal_dg***
Она зависит от трех аргументов: первый аргумент - это коэффицент предпочтения пользователем максимизации площади коттеджа, второй - коэффицент предпочтения им минимизации цены коттеджа, а третий - это таблица (в нашем случае это будет таблица, получившаяся в результате выполнения функции **find_variants**).
Оптимизация по Джеоффриону предполагает следующую взаимозависимость коэффицентов: первый + второй = 1. Смысл данной оптимизации заключается в минимизации суммы отклонений от цели - максимальной площади при минимальной цене. При этом она подстраивается под желания пользователя: с помощью выбора коэффицентов учитывается, какой вариант он предпочитает - большую площадь или меньшую цену.
Формула Джеоффриона реализована следующим образом: в нашей таблице мы создаем новый столбец, который будет содержать значение данной формулы, и вычисляем ее для каждой строки, для этого находим в столбце площади максимальное значенение, а в столбце цены минимальной. Затем для текущей строки берём значение площади, вычитаем его из максимального и умножаем получившейся результат на первый коэффицент, потом берём текущее значение цены, вычитаем из него минимальное и получившийся результат умножаем на второй коэффицент, после чего складываем оба получившихся значения.
Чтобы пользователь смог проверить наши результаты вычислений, мы создаем файл формата json который будет содержать данные по необходимой таблице.
Затем просто находим индекс столбца с минимальным значением получившегося результата и возвращаем информацию о нем в виде таблицы, состоящей из одного столбца.

# Интерфейс
Интерфейс был реализован в виде оконного приложения, для которого использовалась библиотека **trinket**.  

### ***Окно window и вывод информации***
![image](https://user-images.githubusercontent.com/99398012/210432279-efb16e96-59bf-4877-bd33-6183f66e51c7.png)
Элементы в окне расположены в виде таблицы - в первом столбце представлены критерии "Площадь", "Адрес", "Цена", "Отопление", "Электричество" и "Водоснабжение" (за исключением строки Цена/Площадь), во втором - выпадающие списки **ttk.Combobox** c вариантами, сгенерированными моей коллегой. Так, для критерия "Площадь" возможные варианты содержатся в списке sqr_list, для критерия "Адрес" - address_list, и тд. Пользователь задает критерии отбора и нажимает кнопку "Вывести информацию", в результате чего запускается функция **clicked1**, выводящая список коттеджей, удовлетворяющих запрос. В **linfo** записывается результат функции **info** от **find_variants**, аргументами которой выступают выбранные пользователем данные **var**, полученные методом **get()**. Список коттеджей не помещается на экран полностью, поэтому был использован **ttk.Scrollbar** для создания прокрутки. Также для каждого из вариантов коттеджей ставится в соответствие схематичная картинка: для коттеджа создается переменная **numder_floors**, содержащая колтичество этажей, после чего идет сравнение со значениями 1, 2 и 3, и в случае соответствия слева от сгенерированных данных ставится картинка с одно-, двух- или трехэтажным домом таким образом: 
![image](https://user-images.githubusercontent.com/99398012/210435245-397e8f3b-6383-4cdd-83e8-38d38a371885.png)

Однако сначала функция **clicked1** проверяет наличие незаполненных ячеек, и если таковые имеются, то всплывает диалоговое окно.

### ***Диалоговое окно***
Из библиотеки **trinket** использовался **messagebox** для создания диалогового окна. Если есть незаполненные ячейки, то есть метод **get()** от одного из **var** выдает пустую строку, то всплывает сообщение пользователю с требованием заполнить все ячейки.
![image](https://user-images.githubusercontent.com/99398012/210435733-6af08977-1a37-433b-8767-e85389826f31.png)

### ***Выбор оптимального варианта***
Слева от кнопки "Выбрать оптимальный" создан ползунок для выбора соотношения площади и цены с помозью виджета **Scale**, принимающий значения от 0.1 до 0.9 с ходом с 0.1. Пользователь выбирает нужное соотношение и нажимает кнопку, вследствие чего запускается функция **select**, в которой создается окно с единственным оптимальным вариантом коттеджа и его праметрами, который получается из функции **info** от **optimal_dg**
![image](https://user-images.githubusercontent.com/99398012/210437415-83034d0b-b0c9-4786-994c-cefdbf7b7bc2.png)

