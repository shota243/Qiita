# 使用バージョン

- Python 3.7.6
- pytz 2019.3
- tzlocal 1.5.1

# 更新内容

- Python 3.7 で確認
- fromisoformat() メソッドの記述追加

# datetime クラス
Python で日付を扱うクラスは datetime モジュールの datetime クラス。

```py
>>> from datetime import datetime
```

モジュール名とクラス名が同じで紛らわしいため from を使ってクラスのみを import する。
以下 datetime はクラスを指す。
datetime クラスには 2 種類のインスタンスがある。

- aware : タイムゾーン情報を持つ
- naive : タイムゾーン情報を持たない

地域内・短期間における時刻情報を扱うときには暗黙的に同一タイムゾーン内のデータとして扱いタイムゾーンを意識しないことがある。その場合は naive なデータで構わないことになる。
しかし、多国間にまたがるデータや1国内でもアメリカやロシアのように複数のタイムゾーンが存在する国の場合は、タイムゾーンを意識しないと時刻の比較はできない。
また、DST （Daylight Saving Time, 夏時間）が設定されている地域であれば日にちにより時刻がずれるし、同一の地域、同一の日付であっても標準時刻の定義や DST の定義が変遷するために複数年にまたがる時刻データの比較が単純にできない場合もある。
aware なデータを扱うことには必然性があり、また naive で済む用途であっても常に aware なデータを扱うことで覚えておくことを減らすことができる。

## naive インスタンスの生成

datetime クラスのコンストラクタに tzinfo パラメータを渡さなければ naive なインスタンスが作成される。

```py
>>> naivetime = datetime(2018, 1, 2, 3, 4, 5)
naivetime = datetime(2018, 1, 2, 3, 4, 5)
>>> naivetime
naivetime
datetime.datetime(2018, 1, 2, 3, 4, 5)
```

## aware インスタンスの生成

datetime インスタンスを aware にするためにはタイムゾーン情報が必要となる。タイムゾーン情報は datetime モジュールの tzinfo 抽象基底クラスのサブクラスのインスタンスである。

tzinfo の具体的なサブクラスとしては datetime モジュールに timezone クラスが定義されているが、これは UTC からのオフセットを指定して自分でタイムゾーン情報を作るクラスであり、また DST や歴史的な定義の変更に対応していない。同時刻における多タイムゾーン間の比較・変換にのみ使用することができる。

各国政府等により定められた既存のタイムゾーンについては IANA が[タイムゾーンデータベース](https://www.iana.org/time-zones)を管理している。
このデータベースの情報を OS が保持しているものを利用するのが [pytz](https://pypi.org/project/pytz/)モジュールである。pytz は標準モジュールではないため別途インストールが必要である。
また、現在地のタイムゾーン情報を取得するのが [tzlocal](https://pypi.org/project/tzlocal/)モジュールである。tzlocal は pytz で定義されている timezone インスタンスを返す。pytz が必要となる。

```py
>>> from pytz import timezone, utc
>>> from tzlocal import get_localzone
```

pytz モジュールの timezone クラスは datetime モジュールの timezone クラス同様 tzinfo のサブクラスである。両方を使う必要がない限り pytz の timezone だけをそのままの名前で使用する。utc は UTC のタイムゾーンデータ。
また、tzlocal からは get_localzone() 関数のみを import して使用する。

```py
>>> ca = timezone('America/Los_Angeles')
>>> ja = get_localzone()
```

ja は日本のタイムゾーンとなるが、表示すると JST+9:00:00 ではなくLMT+9:19:00 と表示される。同様に ca も LMT-1 day, 16:07:00 となる。

```py
>>> ja
ja
<DstTzInfo 'Asia/Tokyo' LMT+9:19:00 STD>
>>> ca
ca
<DstTzInfo 'America/Los_Angeles' LMT-1 day, 16:07:00 STD>
```

LMT は Local Mean Time の頭文字であり、その場所の平均太陽時。+9:19:00 は東京が東経 139 度 45 扮にあることによる。平均太陽時は統治者の都合により変更されることがない。

datetime クラスのコンストラクタや replace メソッドで tzinfo を指定した場合は LMT が採用されてしまう。astimezone メソッドで UTC に変換してみても 9 時間 19 分が引かれる。

```py
>>> datetime(2018, 1, 2, 3, 4, 5, tzinfo=ja)
datetime(2018, 1, 2, 3, 4, 5, tzinfo=ja)
datetime.datetime(2018, 1, 2, 3, 4, 5, tzinfo=<DstTzInfo 'Asia/Tokyo' LMT+9:19:00 STD>)
>>> naivetime.replace(tzinfo=ja)
naivetime.replace(tzinfo=ja)
datetime.datetime(2018, 1, 2, 3, 4, 5, tzinfo=<DstTzInfo 'Asia/Tokyo' LMT+9:19:00 STD>)
>>> naivetime.replace(tzinfo=ja).astimezone(utc)
naivetime.replace(tzinfo=ja).astimezone(utc)
datetime.datetime(2018, 1, 1, 17, 45, 5, tzinfo=<UTC>)
```

これに対し、timezone クラスの localize メソッドを使うことにより JST aware な datetime インスタンスを得られる。

```py
>>> awaretime = ja.localize(naivetime)
>>> awaretime
awaretime
datetime.datetime(2018, 1, 2, 3, 4, 5, tzinfo=<DstTzInfo 'Asia/Tokyo' JST+9:00:00 STD>)
```

与える naive な datetime を日本で夏時間が施行されていた時期の時刻とすると JDT aware なインスタンスが得られる。

```py
>>> ja.localize(datetime(1950, 7, 2, 3, 4, 5))
ja.localize(datetime(1950, 7, 2, 3, 4, 5))
datetime.datetime(1950, 7, 2, 3, 4, 5, tzinfo=<DstTzInfo 'Asia/Tokyo' JDT+10:00:00 DST>)
```

1950 年 7 月 2 日 3 時 4 分 5 秒から 2018 年 1 月 2 日 3 時 4 分 5 秒までの時間を求めると、naive なデータの場合は 24656 日、東京での aware なデータだと 24656 日と3600 秒（1時間）となる。

```py
>>> naivetime - datetime(1950, 7, 2, 3, 4, 5)
naivetime - datetime(1950, 7, 2, 3, 4, 5)
datetime.timedelta(24656)
>>> awaretime - ja.localize(datetime(1950, 7, 2, 3, 4, 5))
awaretime - ja.localize(datetime(1950, 7, 2, 3, 4, 5))
datetime.timedelta(24656, 3600)
```

この差分を 1950 年 7 月 2 日 3 時 4 分 5 秒に加えると、夏時間のままで計算が行われて元に戻らない。
ただし、== 演算子で比較すると True となる。

```py
>>> ja.localize(datetime(1950, 7, 2, 3, 4, 5)) + (awaretime - ja.localize(datetime(1950, 7, 2, 3, 4, 5)))
ja.localize(datetime(1950, 7, 2, 3, 4, 5)) + (awaretime - ja.localize(datetime(1950, 7, 2, 3, 4, 5)))
datetime.datetime(2018, 1, 2, 4, 4, 5, tzinfo=<DstTzInfo 'Asia/Tokyo' JDT+10:00:00 DST>)
>>> ja.localize(datetime(1950, 7, 2, 3, 4, 5)) + (awaretime - ja.localize(datetime(1950, 7, 2, 3, 4, 5))) == awaretime
<, 5)) + (awaretime - ja.localize(datetime(1950, 7, 2, 3, 4, 5))) == awaretime
True
```

## aware インスタンスのタイムゾーン変換

aware なデータにした後は、astimezone メソッドによりタイムゾーンの変換ができる。表している時刻は変らない。

```py
>>> awaretime.astimezone(utc)
awaretime.astimezone(utc)
datetime.datetime(2018, 1, 1, 18, 4, 5, tzinfo=<UTC>)
```

pytz では、時刻計算は常に UTC にて行い、人が読むための出力のときだけローカルタイムに変関することを推奨している。

> The preferred way of dealing with times is to always work in UTC, converting to localtime only when generating output to be read by humans.

## 文字列へ変換

naive であっても aware であっても datetime クラスの strftime メソッドで文字列に変換できる。

```py
>>> naivetime.strftime('%Y-%m-%dT%H:%M:%S')
naivetime.strftime('%Y-%m-%dT%H:%M:%S')
'2018-01-02T03:04:05'
>>> awaretime.strftime('%Y-%m-%dT%H:%M:%S%z')
awaretime.strftime('%Y-%m-%dT%H:%M:%S%z')
'2018-01-02T03:04:05+0900'
```

naive なインスタンスの場合はタイムゾーン（'%z'）は空文字列となる。

```py
>>> naivetime.strftime('%Y-%m-%dT%H:%M:%S%z')
naivetime.strftime('%Y-%m-%dT%H:%M:%S%z')
'2018-01-02T03:04:05'
```

isoformat メソッドは ISO 8601 形式の文字列を生成する。

```py
>>> naivetime.isoformat()
naivetime.isoformat()
'2018-01-02T03:04:05'
>>> awaretime.isoformat()
awaretime.isoformat()
'2018-01-02T03:04:05+09:00'
```

秒以下の桁の表示はデフォルトではデータに依存するが、timespec キーワードパラメータで指定することもできる。値は 'seconds', 'milliseconds' もしくは 'microseconds'。
また日付けと時刻の間の区切り文字も指定できる。

```py
>>> naivetime.isoformat(' ', timespec='microseconds')
naivetime.isoformat(' ', timespec='microseconds')
'2018-01-02 03:04:05.000000'
```

## 文字列からの変換

文字列から datetime インスタンスを生成するのは strptime クラスメソッドでできる。

```py
>>> datetime.strptime('2018-01-02T03:04:05+0900', '%Y-%m-%dT%H:%M:%S%z')
datetime.strptime('2018-01-02T03:04:05', '%Y-%m-%dT%H:%M:%S')
datetime.datetime(2018, 1, 2, 3, 4, 5)
>>> datetime.strptime('2018-01-02T03:04:05+0900', '%Y-%m-%dT%H:%M:%S%z')
datetime.strptime('2018-01-02T03:04:05+0900', '%Y-%m-%dT%H:%M:%S%z')
datetime.datetime(2018, 1, 2, 3, 4, 5, tzinfo=datetime.timezone(datetime.timedelta(0, 32400)))
```

Python 3.7 から fromisoformat クラスメソッドが使用できる。

```py
>>> datetime.fromisoformat('2018-01-02T03:04:05+09:00')
datetime.fromisoformat('2018-01-02T03:04:05+09:00')
datetime.datetime(2018, 1, 2, 3, 4, 5, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400)))
```

タイムゾーン（'%z'）が無いと naive なインスタンスに、在ると aware なインスタンスになるが、tzinfo は pytz の timezone ではなく datetime の timezone になる。
UTC からのオフセットは情報があるが地域に関する情報はない。
astimezone メソッドで UTC のデータに変敢して処理し、出力時に適切なタイムゾーンに変関することになる。

```py
>>> datetime.strptime('2018-01-02T03:04:05+0900', '%Y-%m-%dT%H:%M:%S%z').astimezone(utc)
datetime.strptime('2018-01-02T03:04:05+0900', '%Y-%m-%dT%H:%M:%S%z').astimezone(utc)
datetime.datetime(2018, 1, 1, 18, 4, 5, tzinfo=<UTC>)
>>> datetime.fromisoformat('2018-01-02T03:04:05').astimezone(utc)
datetime.fromisoformat('2018-01-02T03:04:05').astimezone(utc)
datetime.datetime(2018, 1, 1, 18, 4, 5, tzinfo=<UTC>)
```

## UNIX時間への変換

datetime インスタンスを UNIX 時間に変換するのには timestamp メソッドを使う。float で返るので適宜 int に変換等する。
naive なインスタンスを変関すると、ローカル時間として解釈されて変換される。

```py
>>> naivetime.timestamp()
naivetime.timestamp()
1514829845.0
>>> awaretime.timestamp()
awaretime.timestamp()
1514829845.0
```

## UNIX時間からの変換

UNIX 時間から datetime インスタンスを生成するのには fromtimestamp クラスメソッドを使用する。
tz キーワードパラメータを指定しないと、ローカル時間の naive インスタンスが生成される。

```py
>>> datetime.fromtimestamp(1514829845)
datetime.fromtimestamp(1514829845)
datetime.datetime(2018, 1, 2, 3, 4, 5)
```

tz キーワードパラメータを指定すると、指定したタイムゾーンの aware なインスタンスとなるが、この時 tz に渡すのが pytz の timezone のインスタンスであった場合に、時刻に応じた標準時間なり夏時間なりに設定される。

```py
>>> datetime.fromtimestamp(1514829845, tz=ja)
datetime.fromtimestamp(1514829845, tz=ja)
datetime.datetime(2018, 1, 2, 3, 4, 5, tzinfo=<DstTzInfo 'Asia/Tokyo' JST+9:00:00 STD>)
>>> datetime.fromtimestamp(1514829845, tz=ca)
datetime.fromtimestamp(1514829845, tz=ca)
datetime.datetime(2018, 1, 1, 10, 4, 5, tzinfo=<DstTzInfo 'America/Los_Angeles' PST-1 day, 16:00:00 STD>)
>>> datetime.fromtimestamp(1514829845, tz=utc)
datetime.fromtimestamp(1514829845, tz=utc)
datetime.datetime(2018, 1, 1, 18, 4, 5, tzinfo=<UTC>)
```

utcfromtimestamp クラスメソッドは UTC で表した時刻の naive なインスタンスを生成する。

```py
>>> datetime.utcfromtimestamp(1514829845)
datetime.utcfromtimestamp(1514829845)
datetime.datetime(2018, 1, 1, 18, 4, 5)
```

## 現在時刻の取得

現在時刻を取得するのには now クラスメソッドを使用する。
tz キーワードパラメータを指定しないと、ローカル時間の naive インスタンスが生成される。

```py
>>> datetime.now()
datetime.now()
datetime.datetime(2018, 6, 3, 11, 18, 10, 68268)
```

tz キーワードパラメータを指定すると、指定したタイムゾーンの aware なインスタンスとなるが、この時 tz に渡すのが pytz の timezone のインスタンスであった場合に、時刻に応じた標準耳間なり夏時間なりに設定される。

```py
>>> datetime.now(tz=ja)
datetime.now(tz=ja)
datetime.datetime(2018, 6, 3, 11, 18, 23, 924353, tzinfo=<DstTzInfo 'Asia/Tokyo' JST+9:00:00 STD>)
```

utcnow クラスメソッドは UTC で表した現在時刻の naive なインスタンスを生成する。

```py
>>> datetime.utcnow()
datetime.utcnow()
datetime.datetime(2018, 6, 3, 2, 18, 29, 52492)
```

# まとめ

1. pytz と tzlocal をインストールして使う
1. 時刻を処理するのには datetime モジュールの datetime クラスで UTC 時刻の aware なインスタンスを使う。
1. 人間が読むための出力時に astimezone メソッドでタイムゾーンを変換する。
1. タイムゾーンを表すのは pytz モジュールの timezone クラス。
1. datetime のコンストラクタではタイムゾーンを指定せず、一旦 naive なインスタンスを作ってから pytz の timezone の localize メソッドで aware なインスタンスを生成する。
1. 文字列への変換は strftime メソッドもしくは isoformat メソッド。ifoformat メソッドでは timespec を指定する。
1. 文字列からの変換は strptime クラスメソッドもしくは fromisoformat クラスメソッド。タイムゾーンがない場合は naive なインスタンスになるので localize でローカル時刻の時刻データに変換後、さらに UTC の時刻に変換する。
1. UNIX 時刻への変換は timestamp メソッド。
1. UNIX 時刻からの変換は fromtimestamp クラスメソッド。tz パラメータでタイムゾーン pytz.urc を指定する。
1. 現在時刻の取得は now クラスメソッド。tz パラメータでタイムゾーン pytz.urc を指定する。
1. ローカルのタイムゾーン取得は tzlocal.get_localzone 関数。

```py
from datetime import datetime
from pytz import timezone, utc
from tzlocal import get_localzone

def naive_to_utc(dt):
    return get_localzone().localize(dt).astimezone(utc)

def datetimeutc(year, month, day, hour=0, minute=0, second=0, microsecond=0, fold=0):
    naivetime = datetime(year, month, day, hour=hour, minute=minute, second=second,
                         microsecond=microsecond, fold=fold)
    return naive_to_utc(naivetime)

def isoformatlocal(dt, tz=None, timespec='seconds'):
    if tz is None:
       tz = get_localzone()
    return dt.astimezone(tz).isoformat(timespec=timespec)

def strptimeutc(date_string, format):
    if '%z' in format:
        return datetime.strptime(date_string, format).astimezone(utc)
    else:
        return naive_to_utc(datetime.strptime(date_string, format))

def nowutc():
    return datetime.now(tz=utc)
```
