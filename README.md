**КОД ПРОГРАММЫ:**

import os
import shutil
import subprocess

# Функция создания файла
def create_file():
    try:
        file_path = input("Введите путь и имя файла для создания: ")
        if file_path:
            with open(file_path, 'w') as file:
                print(f"Файл {file_path} создан")
                git_operations("add", None, None)
        else:
            print("Неверно введен путь и имя файла")
    except Exception as e:
        print(f"Ошибка при создании файла: {e}")
    main()

# Функция удаления файла или папки
def delete_item():
    item = input("Введите путь и имя файла или папки для удаления: ")
    if item:
        if os.path.exists(item):
            if os.path.isfile(item):
                os.remove(item)
                print(f"Файл {item} успешно удален")
                git_operations("rm", item, None)
            elif os.path.isdir(item):
                shutil.rmtree(item)
                print(f"Папка {item} успешно удалена")
                git_operations("rm", item, None)
        else:
            print(f"Файл или папка {item} не существует")
    main()

# Функция перемещения файла или папки
def move_item():
    old_path = input("Введите путь и имя файла или папки для перемещения: ")
    if old_path:
        new_path = input("Введите новый путь: ")
        if new_path:
            if os.path.exists(old_path):
                if os.path.exists(new_path):
                    if (new_path == old_path):
                        print("Директории файлов совпадают")
                    else:
                        shutil.move(old_path, new_path)
                        print(f"Файл или папка {old_path} успешно перемещен(а) в {new_path}")
                        git_operations("mv", old_path, new_path)
                else:
                    print(f"Файл или папка {new_path} не существует")
            else:
                print(f"Файл или папка {old_path} не существует")
    main()

# Функция копирования файла
def copy_file():
    old_file = input("Введите путь и имя файла для копирования: ")
    if old_file:
        new_file = input("Введите путь и имя нового файла: ")
        if new_file:
            if os.path.exists(old_file):
                shutil.copy2(old_file, new_file)
                print(f"Файл {old_file} успешно скопирован в {new_file}")
                git_operations("cp", old_file, new_file)
            else:
                print(f"Файл {old_file} не существует")
    main()

# Функция работы с git
def git_operations(command, old_loc, new_loc):
    repo_path = "files"
    # Создаем репозиторий git, если он еще не был создан
    if not os.path.exists(repo_path):
        os.makedirs(repo_path)
    os.chdir(repo_path)
    subprocess.run(["git", "init"])

    if command == "add" or command =="cp":
        # Добавляем файлы в репозиторий и делаем коммит
        subprocess.run(["git", "add", "."])
    elif command == "rm":
         # Удаляем файлы из репозитория и делаем коммит
        subprocess.run(["git", "rm", old_loc])
    elif command =="mv":
        # Перемещаем/удаляем файлы в репозиторий и делаем коммит
        subprocess.run(["git", "add", new_loc])
        subprocess.run(["git", "rm", old_loc])

    name = input("Введите название для комита: ")
    subprocess.run(["git", "commit", "-m", name])

    # Привязываем локальный репозиторий к удаленному и загружаем изменения
    remote_url = "https://github.com/Katerinahar/files.git" # URL удаленного репозитория
    result = subprocess.run(["git", "remote", "get-url", "origin"], capture_output=True) 
    if result.returncode != 0: # Проверяем существует ли ветка origin, если нет - создаем ее
        subprocess.run(["git", "remote", "add", "origin", remote_url])
 
    subprocess.run(["git", "pull", "origin", "master", "--allow-unrelated-histories"])  # Добавляем эту строку для слияния удаленных изменений и добавляем флаг для разрешения несвязанных историй
    if command != "rm":
        subprocess.run(["git", "push", "origin", "--delete", old_loc])  # Удаляем файл из удаленного репозитория
        subprocess.run(["git", "push", "-u", "origin", "master", "--force"])
    else: subprocess.run(["git", "push", "-u", "origin", "master"])

def main():
    # Выбор операции
    operation = input("Выберите операцию: \n 1. Создать файл \n 2. Удалить файл или папку \n 3. Переместить файл или папку \n 4. Скопировать файл \n")

    # Выполнение операции
    if operation == "1":
        create_file()
    elif operation == "2":
        delete_item()
    elif operation == "3":
        move_item()
    elif operation == "4":
        copy_file()
    else:
        print("Ошибка ввода")
        main()
main()
