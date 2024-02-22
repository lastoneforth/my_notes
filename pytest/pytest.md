
```bash
py -m ensurepip --upgrade
```

# 测试函数

## assert 判断

## pytest.fail()

```python
def test_with_fail():
    c1 = Card("sit there", "brian")
    c2 = Card("do something", "okken")
    if c1 != c2:
        pytest.fail("they don't match")
```

## pytest.raises()

用于应对预期会触发异常的测试  

```python

# 测试会显示FAIL,TypeError，缺少参数
def test_no_path_raises():
    cards.CardsDB()

# 测试会PASS
def test_no_path_raises():
    with pytest.raises(TypeError):
        cards.CardsDB()

```

## 内置mark

```python
@pytest.mark.skip(reason="misunderstood the api)


@pytest.mark.skipif(
    tasks.__version__ < "0.2.0">
    reason="not supported unitl version 0.2.0"
)

@pytest.mark.xfail(
    tasks.__version__ < "0.2.0">
    reason="not supported unitl version 0.2.0"
)

// 对于xfail标记的用例，预期结果是失败
// 如果实际用例执行失败了，显示小写 x
// 如果实际用例执行成功了，显示大写 X
```

## 运行测试子集

| 测试子集 | 方法 |
| :--: | :--: |
| 单个测试文件/模块 | pytest 路径名+文件名 |
| 单个测试函数 | pytest 文件名::函数名 |
| 单个测试类 | pytest 文件名::类名 |
| 单个测试类中的特定测试方法 | pytest 文件名::类名::方法名 |

  另外，可以使用 -k <表达式> 方式对测试用例进行筛选  

```bash
pytest -v -k "_raises and not delete"

// 选取方法名中包含 _raises 但是不包含 delete 的方法执行
```

## 参数化测试

使用 @pytest.mark.parametrize(argnames, argvalues) 给测试方法批量传输数据  

```python
@pytest.mark.parametrize('task',
                         [
                             Task('sleep', done=True),
                             Task('wake', 'brian'),
                             Task('breathe', 'BRIAN', True),
                             Task('exercise', 'BrIaN', False)
                         ])
def test_add_2(task):
    task_id = tasks.add(task)
    t_from_db = tasks.get(task_id)
    assert equivalent(t_from_db, task)
```

## 测试方法的结构

Arrange-Act-Assert模式（或者称为 Given-When-Then模式）  
将测试方法分为三个阶段：

- getting ready to do something
- doing something
- checking to see if it worked

# pytest fixtures

fixture是什么：

- fixture是方法
- pytest在测试方法运行之前或者之后运行的方法

## 基本使用方法

- 使用 pytest.fixture()装饰一个方法
- 在测试方法的参数列表中传入fixture的方法名
- pytest在运行测试方法前（或后）会运行fixture对应的方法

```python
import pytest

@pytest.fixture()
def some_data():
    """Return answer to ultimate question"""
    return 42

def test_some_data(some_data):
    """Use fixture return value in a test"""
    assert some_data == 42
```

测试方法过程中出现exception时，结果会显示 FAIL  
fixture过程中出现exception时，结果会显示 Error  

### yield

fixture中yield之前的部分会在测试函数之前运行，yield之后的部分会在测试函数之后运行

```python
import pytest
from pathlib import Path
from tempfile import TemporaryDirectory
import cards


@pytest.fixture()
def cards_db():
    with TemporaryDirectory() as db_dir:
        db_path = Path(db_dir)
        db = cards.CardsDB(db_path)
        yield db
        db.close()


def test_empty(cards_db):
    assert cards_db.count() == 0
```

### pytest  --setup-show参数

运行时加上 --setup-show 参数，可以看到test function和fixture的运行顺序  

### fixture scope 设置

| scope参数设置 | 作用范围 |
| :--: | :--: |
| scope="function" | run once per test function，默认取值 |
| scope="class" | run once per test class |
| scope="module" | run once per module |
| scope="package" | run once per package or test directory |
| scope="session" | run once per session |

在 test module 中定义 fixture 时，seesion和package 作用范围和 module 表现一样，如果想要作用于其他 scope，需要在 conftest.py 中定义  

### 通过 conftest.py 共享 fixture

- 创建 conftest.py
- 在 conftest.py 中定义 fixture
- 在测试文件中,测试方法的参数列表中直接使用fixture

pytest 会自动读取 conftest.py 文件，不需要在其他文件中 import conftest  

### 查看 fixture 定义位置

- pytest --fixtures <test_range>
- pytest --fixtures-per-test <test_range>

### fixture 的嵌套

fixture 的方法的参数列表中可以使用其他 fixture ，
这里其他的fixture的作用范围不能低于当前的fixture  

```python
@pytest.fixture(scope="session")
def db():
    """CardsDB objects connected to a temporary database"""
    with TemporaryDirectory() as db_dir:
        db_path = Path(db_dir)
        db_ = cards.CardsDB(db_path)
        yield db_
        db_.close()


@pytest.fixture(scope="function")
def cards_db(db):
    """CardsDB object that's empty"""
    db.delete_all()
    return db
```

### 动态决定 fixture 的 scope 

```python
def db_scope(fixture_name, config):
    if config.getoption("--func-db", None):
        return "function"
    return "session"


def pytest_addoption(parser):
    parser.addoption(
        "--func-db",
        action="store_true",
        default=False,
        help="new db for each test"
    )

# 这里 scope的取值，根据 db_scope()方法的返回值决定
# db_scope()的返回值根据 pytest 运行参数确定
# 当有参数 --func-db 时，scope取值为 function
# 没有参数 --func-db 时，scope取值为 session
@pytest.fixture(scope=db_scope)
def db():
    """CardsDB objects connected to a temporary database"""
    with TemporaryDirectory() as db_dir:
        db_path = Path(db_dir)
        db_ = cards.CardsDB(db_path)
        yield db_
        db_.close()
```

### autouse 参数

```python
# autouse=True 该fixture会自动应用到测试中
@pytest.fixture(autouse=True)
```

### fixture 重命名

使用 name 参数对fixture进行重命名  

```python
@pytest.fixture(name="ultimate_answer")
def ultimate_answer_fixture():
    return 42

def test_everything(ultimate_answer):
    assert ultimate_answer==42
```

# 内置 fixtures

## tmp_path 和 tmp_path_factory  

- tmp_path : function-scope , 返回一个 pathlib.Path 实例，该实例指向一个临时目录
- tmp_path_factory : session-scope ， 返回一个 TempPathFactory 对象，通过调用 mktemp() 方法返回一个 Path 对象  

```python
def test_tmp_path(tmp_path):
    file = tmp_path/"file.txt"
    file.write_text("Hello")
    assert file.read_text() == "Hello"


def test_tmp_path_factory(tmp_path_factory):
    path = tmp_path_factory.mktemp("sub")
    file = path/"file.txt"
    file.write_text("Hello")
    assert file.read_text() == "Hello"
```

## capsys

用于捕获输出到 stdout， stderr 的内容  

```python

#1 捕获应用程序的输出

# 通过 capsys 捕获
# capsys.readouterr() 返回一个 namedtuple, 有 out 和 err 两个属性
# rstrip() 去除新行
def test_version_v2(capsys):
    cards.cli.version()  # 这里会在控制台产生输出
    output = capsys.readouterr().out.rstrip()
    assert output == cards.__version__

# --------------------------------------

#2 让测试代码中的print可以在pytest中正常显示
def test_disabled(capsys):
    with capsys.disabled():
        print("\ncapsys disabled print")
```

## monkeypatch

可以 fake 应用程序的输出  

# Parametrization 参数化

三种方法

- parametrizing functions
- parametrizing fixtures
- using a hook function called <i>pytest_generate_tests<i>

## Parametrizing functions

使用 @pytest.mark.parametrize() 装饰器  

- () 内给出测试的参数，和参数的值
- 测试的参数以双引号 "" 包围，多个参数之间使用逗号隔开
- 参数的值以列表形式给出，以方括号 [] 包围，多个参数的一组取值以元组形式给出
- 测试方式的参数列表中需要有 @pytest.mark.parametrize() 中给出的测试参数

## Parametrizing fixtures

把参数列表放在fixture中

```python
# 下面的request 和 request.param 是固定写法， request 是 pytest 内置的一个fixture
@pytest.fixture(params=["done", "in prog", "todo"])
def start_state(request):
    return request.param


def test_finish(cards_db, start_state):
    c = Card("write a book", state=start_state)
    index = cards_db.add_card(c)
    cards_db.finish(index)
    card = cards_db.get_card(index)
    assert card.state == "done"
```

对于fixture传入多个参数

```python
data = [{'user': "anjing",
         'pwd': "123456",},
        {'user': "admin",
         "pwd": "123"},
        {'user':"test",
         'pwd': '1234'}]


@pytest.fixture(params=data)
def login(request):
    print('登录功能')
    yield request.param
    print('退出登录')

class Test_01:
    def test_01(self, login):
        print('---用例01---')
        print('登录的用户名：%s' % login['user'])
        print('登录的密码：%s' % login['pwd'])

    def test_02(self):
        print('---用例02---')
```

通过和ids参数配合，可以让pytest输出可读性更强

```python
data = [{"start_summary": "write a book",
         "start_state": "done"},
        {"start_summary": "second edition",
         "start_state": "in prog"},
        {"start_summary": "create a course",
         "start_state": "todo"},
        ]

data_mark = ["write a book & done",
             "second edition & in prog",
             "create a course & todo"]


@pytest.fixture(params=data, ids=data_mark)
def combined_start(request):
    yield request.param


def test_finish_combined(cards_db, combined_start):
    c = Card(summary=combined_start["start_summary"],
             state=combined_start["start_state"])
    index = cards_db.add_card(c)
    cards_db.finish(index)
    card = cards_db.get_card(index)
    assert card.state == "done"
```

## Parametrizing with pytest_generate_tests

```python
def pytest_generate_tests(metafunc):
    # 这里是判断 start_state 和 start_summary 是否在测试方法使用的 fixture 中（是否在参数列表中）
    # 如果是的话，就对这两个进行参数化
    if "start_state" in metafunc.fixturenames and "start_summary" in metafunc.fixturenames:
        metafunc.parametrize("start_summary, start_state",
                             [("write a book", "done"),
                              ("second edition", "in prog"),
                              ("create a course", "todo")])


def test_finish(cards_db, start_summary, start_state):
    c = Card(summary=start_summary, state=start_state)
    index = cards_db.add_card(c)
    cards_db.finish(index)
    card = cards_db.get_card(index)
    assert card.state == "done"

```
