# intro
- python packaging & distribution test 
- test on `bdist`, `sdist`, `bdist_wheel` 

# steps

1) mkdir + bootstrap

```bash
$ mkdir calc_john
$ cd calc_john
$ virtualenv env
```
2) write `setup.py`, `LICENSE`, `README.md`

```py
# setup.py

import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="calc-john",
    version="0.0.2",
    author="Jongsuk Oh",
    author_email="aaa@abc.com",
    description="A small calculator",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/fransoaardi/calc_john",
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires='>=3.6',
)
```

> dir is like below
```bash
$ tree -L 1
.
├── LICENSE
├── README.md
├── calc_john
├── env
└── setup.py
```

3) install `twine`, `setuptools`, `wheel`
```bash
$ ./env/bin/pip3 install twine setuptools wheel
```

4) packaging
```bash
$ ./env/bin/python3 setup.py sdist bdist_wheel

$ tree -L 1
.
├── LICENSE
├── README.md
├── build
├── calc_john
├── calc_john.egg-info
├── dist
├── env
└── setup.py
```

5) upload to repostiory (test-pypi)
```bash
$ ./env/bin/python3 -m twine upload --repository-url https://test.pypi.org/legacy/ dist/*
```


---

# how to install and utilize

1) mkdir test-dir

```bash
$ mkdir test-dir
$ cd test-dir
$ virtualenv env
```

2) install package

```
$ ./env/bin/pip3 install --index-url https://test.pypi.org/simple/ --no-deps calc_john
```

3) main.py
```python
from calc_john.calculator import Calculator

if __name__ == "__main__":
    calc = Calculator()
    print(calc.result)
    calc.add(3)
    print(calc.result)
```

4) run main.py
```
$ ./env/bin/python3 main.py
0
3
```

# tests on cross-platform 

- bdist(binary distribution), sdist(source distribution), bdist_wheel(wheel) 옵션을 줄 수 있는데, 각기 어떻게 빌드되는지 테스트해보았다. 

- `bdist`, `sdist`, `bdist_wheel`, `bdist bdist_wheel`, `sdist bdist_wheel` 테스트 해봤는데, 결국 `dist/*` dir 의 내용을 `twine` 으로 `pypi` 로 업로드 하게 될것이다. 

- `bdist` 는 platform dependent 한 `.tar.gz` 를, `sdist` 는 그냥 `.tar.gz` 를, `bdist_wheel` 은 `.whl` 파일을 만들었다. 
- `bdist_wheel` 은 항상 cross-platform 한 파일을 만들어내지 않는다, [PEP425](https://www.python.org/dev/peps/pep-0425/) 에 따르면 wheel 이 만들어 내는 naming 규칙이 있고, python2,3 / abi / platform 태그가 붙는다. 
- 아래에서 만들어낸 `calc_john-0.0.2-py3-none-any.whl` 는 python3로 빌드, abi 상관없고, platform 상관없기때문에 그렇다. (setup.py 빌드 당시, pure python code 로 판단되었기 때문인것으로 예상된다)

- python3, python2 로 `bdist_wheel` 옵션 각각 빌드하면, `calc_john-0.0.2-py2-none-any.whl`, `calc_john-0.0.2-py3-none-any.whl` 만들어질거라 pip, pip3 에 따라 다운될 .whl 을 둘다 만들게되는 것이다. 


## 결론

- `sdist` `bdist_wheel` 옵션 같이쓰면 큰 문제 없다 
- `.whl` 파일은 항상 cross-platform 은 아니다 
- `sdist bdist_wheel` 로 build, distribute 했다고 가정하면, `.whl` 은 binary 이기 때문에, `pip` 가 현재 platform 에 맞는 `.whl` 을 찾으면 다운로드 받을것이고, 맞지 않다면 source 로 fallback 될 것이다

### reference

- https://packaging.python.org/guides/distributing-packages-using-setuptools/#wheels
- https://packaging.python.org/tutorials/packaging-projects/#generating-distribution-archives

## bdist
```bash
$ python3 setup.py bdist
$ tree -L 4 -I env
.
├── LICENSE
├── README.md
├── build
│   ├── bdist.macosx-10.15-x86_64
│   └── lib
│       └── calc_john
│           ├── __init__.py
│           └── calculator.py
├── calc_john
│   ├── __init__.py
│   └── calculator.py
├── calc_john.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   └── top_level.txt
├── dist
│   └── calc-john-0.0.2.macosx-10.15-x86_64.tar.gz
└── setup.py
```

## sdist
```bash
$ python3 setup.py sdist
$ tree -L 4 -I env
.
├── LICENSE
├── README.md
├── calc_john
│   ├── __init__.py
│   └── calculator.py
├── calc_john.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   └── top_level.txt
├── dist
│   └── calc-john-0.0.2.tar.gz
└── setup.py
```

## bdist_wheel
```bash
$ python3 setup.py bdist_wheel
$ tree -L 4 -I env
.
├── LICENSE
├── README.md
├── build
│   ├── bdist.macosx-10.15-x86_64
│   └── lib
│       └── calc_john
│           ├── __init__.py
│           └── calculator.py
├── calc_john
│   ├── __init__.py
│   └── calculator.py
├── calc_john.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   └── top_level.txt
├── dist
│   └── calc_john-0.0.2-py3-none-any.whl
└── setup.py
```

## bdist bdist_wheel
```bash
$ python setup.py bdist bdist_wheel
$ tree -L 4 -I env
.
├── LICENSE
├── README.md
├── build
│   ├── bdist.macosx-10.14-x86_64
│   └── lib
│       └── calc_john
│           ├── __init__.py
│           └── calculator.py
├── calc_john
│   ├── __init__.py
│   └── calculator.py
├── calc_john.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   └── top_level.txt
├── dist
│   ├── calc-john-0.0.2.macosx-10.14-x86_64.tar.gz
│   └── calc_john-0.0.2-py3-none-any.whl
└── setup.py
```

## sdist bdist_wheel
```bash
$ python3 setup.py sdist bdist_wheel
$ tree -L 4 -I env
.
├── LICENSE
├── README.md
├── build
│   ├── bdist.macosx-10.15-x86_64
│   └── lib
│       └── calc_john
│           ├── __init__.py
│           └── calculator.py
├── calc_john
│   ├── __init__.py
│   └── calculator.py
├── calc_john.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   └── top_level.txt
├── dist
│   ├── calc-john-0.0.2.tar.gz
│   └── calc_john-0.0.2-py3-none-any.whl
└── setup.py
```


# reference

- https://packaging.python.org/tutorials/packaging-projects/
