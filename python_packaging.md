# intro
python packaging & distribution test 

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

# reference

- https://packaging.python.org/tutorials/packaging-projects/
