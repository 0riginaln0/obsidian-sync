# Введение
Буду краток, Obsidian - лучшая альтернатива Ноушену, которая есть на данный момент. 
С тех пор как в России ноушн прекратил работу я перешёл на Обсидиан и получаю тот же опыт, разве что без веб версии приложения.

Я настроил синхронизацию через гугл диск. На нём, как я знаю, всем дают 15гб, и этого вполне хватит для текстовых файлов.
# Настройка синхронизации
По итогу у меня:
- на Windows синхронизация работает в автоматическом режиме
- на Android в полуавтоматическом режиме
- на Linux в ручном режиме

В самом Гугл Диске создал папку Vaults в которой будут лежать все хранилища Obsidian. У меня пока только одно хранилище - Main. Поэтому в Гугл Драйве структура будет такая `Vaults/Main`. 
## Windows
Скачал [Google Drive For Desktop](https://support.google.com/drive/answer/10838124?hl=en)
[Туториал как настроить эту программу](https://youtu.be/26PKoz3yb0M?si=zc3H4xqctV0H6mHG)
Подключил свой гугл аккаунт.
Все настройки можно пропускать, так как они нам не нужны.

А дальше надо эту папку Vaults сделать доступной оффлайн. Для этого входим в проводник, в диск гугл драйва, щёлкаем правой кнопкой мышки на `Vaults` -> `Оффлайн-доступ` -> `Оффлайн-доступ включён`
Всё. Теперь можно создавать в папке Vaults свои хранилища из Obsidian и они сами будут синхронизироваться в автоматическом режиме.

# Android
Скачал приложение FolderSync от Tacit Dynamics
Делал всё по этому [видео-туториалу](https://youtu.be/0LZSFvyCmEk?si=IxG-t7yAbtnKdu4p) . Интерфейс немного изменился, но думаю проблем понять что где не будет.

Там всё просто. Главное только чтобы вы выбрали `Two-way` соединение. И правильно выбрали папку `Vaults` гугл драйва и папку на телефоне с которой будет всё синхронизироваться. И ещё поставить галочку *"Синхронизировать удаления"*. В настройках синхронизации можно выбрать как часто приложение будет само синхронизироваться. Я выбрал один раз в 12 часов. Чаще мне не надо, потому что мне не сложно зайти в FolderSync и прожать кнопку синхронизации до и после работы в Obsidian.

# Linux
Что-ж мои дорогие красноглазики :)

Устанавливаем [Rclone](https://rclone.org/)
Эта утилита позволит нам синхронизировать локальную папку с папкой из гугл диска.
По Rclone есть [туториал на русском](https://www.youtube.com/watch?v=qKw8pNC_dt8&t=605s). Чисто понять что да как. 
По туториалу добавляем коннекшн к Google Drive. GOOGLE DRIVE. google drive. (не перепутайте с другими облачными хранилищами)
В процессе можете сверяться с [официальным туториалом](https://rclone.org/drive/).

Как коннекшн готов, нам нужна только одна команда:
`rclone sync`

Чтобы не прописывать её в ручную я написал Lua скрипт, который будет делать это за меня.
```lua
#!/usr/local/bin/lua

local local_folder = "/home/ryzh/Downloads/Programs/Obsidian/Vaults"
local remote_folder = "Vaults"
local conn = "Drive:"

---Копирует содержимое папки `source` в папку `destination`
---Пушит изменения в Google Drive
local function sync_push(source, connection, destination)
    local cmd = "rclone sync " ..
        source .. " " ..
        connection .. destination
    return os.execute(cmd)
end

---Копирует содержимое папки `source` в папку `destination`
---Пуллит изменения из Google Drive
local function sync_pull(source, connection, destination)
    local cmd = "rclone sync " ..
        connection .. source ..
        " " .. destination
    return os.execute(cmd)
end

print("Hello from syncing program!")
local command = arg[1]
if (command ~= "pull") and (command ~= "push") then
    print("Usage:")
    print([[
    for getting changes from the google drive:
        sync-vaults pull
    for submitting changes to google drive:
        sync-vaults push
]])
else
    if command == "push" then
        print("Running push command")
        print(sync_push(local_folder, conn, remote_folder))
    elseif command == "pull" then
        print("Running pull command")
        print(sync_pull(remote_folder, conn, local_folder))
    end
end
```
Имя у этого файла `sync-vaults` и важно добавить права на исполнение для этого файла: `chmod +x sync-vaults`.
 И ещё я добавил путь до папки в которой он находится в `.bashrc`:
	`export PATH=$PATH:/home/ryzh/Downloads/Programs/Obsidian`
В итоге в любом месте я могу писать `sync-vaults pull`, чтобы стянуть последние изменения с гугл диска, и `sync-vaults push`, чтобы отправить изменения.
