## Install
Clone the repository to somewhere, navigate into `cpp_wrapper` and enter:

```
pip install -e .
```

This will install the modules into your python environment.

## Create a New Project
Navigate to somewhere you wanna create a new project in and enter:

```
create_project.py -p `your_project_name` -a `your_app_name`
```

For simplicity, assume your project name is `project` and your app name is `apps`.

This will create a folder `project` in the directory where you issue the command above. The content of the directory is like:

```
  project
  |__ CMakeLists.txt
  |__ README.md
  |__ .gitignore
  |__ LICENCE.md
  |
  |__ apps
  |   |__ app.cpp
  |   |__ CMakeLists.txt
  |
  |__ cmake
  |   |__ FindSomeLib.cmake
  |
  |__ includes
  |   |__ project
  |       |__ lib.cpp
  |
  |__ src
  |   |__ CMakeLists.txt
  |   |__ lib.cpp
  |
  |__ tests
  |   |__ CMakeLists.txt
  |   |__ testlib.cpp
  |
  |__ docs
  |   |__ CMakeLists.txt
  |
  |__ scripts
  |   |__ helper.py
```

