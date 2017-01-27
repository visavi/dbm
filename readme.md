#Класс для работы с базой данных

```
DBM::run()->config(
    'localhost',
    'database',
    'username',
    'password',
    'port'
);
```

<h3>count(string $table, array $params = null)</h3>
<b>Получение количества элементов в таблице</b><br />

Принимает имя таблицы и массив параметров<br />

Выводит количество всех сообщений в гостевой книге
<pre class="prettyprint linenums">
$total = DBM::run()->count('guest');
</pre>

Выводит количество всех сообщений пользователя Vantuz в гостевой книге
<pre class="prettyprint linenums">
$total = DBM::run()->count('guest', array('user' => 'Vantuz'));
</pre>

Выводит количество сообщений в гостевой книге за последний час
<pre class="prettyprint linenums">
$total = DBM::run()->count('guest', array('time' => array('>', SITETIME - 3600)));
</pre>

<h3>select(string $table, array $params = null, int $limit = null, int $start = null, array $order_by = null)</h3>
<b>Получение записей из таблицы</b><br />

Принимает имя таблицы, массив параметров, количество возвращаемых элементов, смещение и сортировка<br />

Получает 10 записей из гостевой книги со смещением 50 записей отсортированных по time по убыванию
<pre class="prettyprint linenums">
$posts = DBM::run()->select('guest', null, 10, 50, array('time'=>'DESC'));
</pre>

Получает все записи из гостевой книги отсортированных по time по убыванию
<pre class="prettyprint linenums">
$posts = DBM::run()->select('guest', null, null, null, array('time'=>'ASC'));
</pre>

Получает все записи пользователя Vantuz из гостевой книги за последний час
<pre class="prettyprint linenums">
$posts = DBM::run()->select('guest', array('user' => 'Vantuz', 'time' => array('>', SITETIME - 3600)));
</pre>

Просто получает все записи из гостевой книги
<pre class="prettyprint linenums">
$posts = DBM::run()->select('guest');
</pre>

<h3>selectFirst(string $table, array $params = array(), array $order_by = null</h3>
<b>Получает первую запись из запроса</b><br />

Аналогична выражению
<pre class="prettyprint linenums">
select($table, $params, 1, null, $order_by)
</pre>

<h3>delete($table, $params = array())</h3>
<b>Удаляет записи из БД, возврашает количество затронутых элементов</b><br />

Удаляет запись из гостевой книги с ID = 55
<pre class="prettyprint linenums">
$delete = DBM::run()->delete('guest', array('id' => 55));
</pre>

Удаляет записи из гостевой книги старее чем 1 месяц (3600 (секунды в часе) * 24 часа * 30 дней)
<pre class="prettyprint linenums">
$delete = DBM::run()->delete('guest', array('time' => array('<', SITETIME - 3600 * 24 * 30)));
</pre>

<h3>update(string $table, array $params, array $wheres = array(), boolean $timestamp_this = false)</h3>
<b>Обновляет записи в БД</b><br />
Возвращает количество затронутых элементов или Exception в случае ошибки<br />
Если передан $timestamp_this, автоматически обновится поле modified на текущий timestamp<br /><br />

Обновляет текст записи в гостевой книге с ID под номером 5
<pre class="prettyprint linenums">
$update = DBM::run()->update('guestbook', array(
	'text' => 'Новый текст',
), array(
	'id' => 5
));
</pre>

Добавляет пользователю Vantuz 1 балл и 5 монет
<pre class="prettyprint linenums">
$user = DBM::run()->update('users', array(
	'point'    => array('+', 1),
	'money'    => array('+', 5),
), array(
	'login' => 'Vantuz'
));
</pre>

<h3>insert(string $table, array $params = array(), boolean $timestamp_this = false)</h3>
<b>Добавляет запись в БД</b><br />
Возвращает ID вставленной записи в случае успеха или Exception<br />

Если передан $timestamp_this, автоматически установится поле modified и created с текущим timestamp<br /><br />

Добавление записи в гостевую книгу
<pre class="prettyprint linenums">
$guest = DBM::run()->insert('guest', array(
	'user' => $log,
	'text' => $msg,
	'ip'   => $ip,
	'brow' => $brow,
	'time' => SITETIME,
));
</pre>

<h3>insertMultiple(string $table, array $columns = array(), array $rows = array(), boolean $timestamp_these = false)</h3>
<b>Добавляет несколько записей в таблицу с помощью одного запроса</b><br />
Возвращает true в случае успеха или делает откат транзакции и возвращает Exception<br />

<pre class="prettyprint linenums">
DBM::run()->insertMultiple('guest', array(
		'user', 'text', 'ip', 'brow', 'time'
	),
	array(
		array('Vantuz', 'Сообщение первое', '123.222.111.33', 'Opera', SITETIME),
		array('Vantuz', 'Сообщение второе', '123.222.111.33', 'Chrome', SITETIME),
	)
);
</pre>

<h3>execute(string $query, array $params = array())</h3>
<b>Выполняет подготовленный запрос</b><br />
Возвращает количество затронутых элементов или Exception

<pre class="prettyprint linenums">
DBM::run()->execute("DELETE FROM `guest` WHERE `id`=:id;", array('id' => 5));
</pre>

<h3>query(string $query, array $params = array())</h3>
<b>Выполняет подготовленный запрос</b><br />
Возвращает набор данных в виде массива подходящий по запросу или Exception

<pre class="prettyprint linenums">
$posts = DBM::run()->query("SELECT * FROM guest WHERE user = :user LIMIT :limit;",
	array(
		'user'  => 'Vantuz',
		'limit' => 10,
	)
);
</pre>


<h3>queryFirst(string $query, array $params = array())</h3>
<b>Выполняет подготовленный запрос</b><br />
Возвращает первый элемент массива найденных данных или Exception

<pre class="prettyprint linenums">
$post_id = DBM::run()->queryFirst("SELECT `id` FROM `guest` WHERE `user`=:user AND `id`<>:id LIMIT 1;", compact('user', 'id'));
</pre>
