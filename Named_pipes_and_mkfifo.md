
Одна из концепций, которые мы будем использовать в этом гайде, это именованные пайпы (named pipes).

Помните в bash можно передать stdout одной программы в stdin другой через `cmd1 | cmd2` ? Но что, если cmd1 и cmd2 запущены в разных терминалах, или мы хотим запустить cmd2 раньше, чем cmd1? В этом случае нам как раз пригодятся named pipes.

Создать named pipe очень просто: `mkfifo /path/to/file`. Полученный файл на самом деле не является файлом на диске (как unix domain socket) и чтение/запись в него не включает в себя I/O операции на диске.

# Использование

Сначала открываете fifo на чтение, затем на запись. Например:

```bash
mkfifo /tmp/myfifo
tail -f /tmp/myfifo # в соседнем терминале
echo "Hello" > /tmp/myfifo
echo "World" > /tmp/myfifo
long_running_program 1> /tmp/myfifo 2>&1 # перенаправить и stdout и stderr
```

И в выводе tail -f вы увидите вывод программ.

**ОЧЕНЬ ВАЖНАЯ ИНФОРМАЦИЯ КОТОРУЮ НУЖНО ЗАПОМНИТЬ!!!**

Когда вы делаете `echo "Hello" > /tmp/myfifo`, вы сначала **открываете** fifo на запись, затем **пишете**. По умолчанию bash, когда вы используете `>`, открывает fifo или любой другой файл в блокирующем режиме (процесс приостанавливает свое выполнение (переходит в статус **S**leeping) до того как сможет открыть файл). В случае с fifo, операция открытия блокируется до тех пор, пока fifo не будет открыт "другим концом" на чтение, после чего процесс продолжит работу (перейдет в **R**unning).

Однако попытка **записать** (в уже открытый) fifo, если он не открыт "на другом конце" в режиме чтения, приведет к тому, что пишущий процесс получит SIGPIPE. Этот сигнал завершает процесс, если его не обрабатывать явно. **И это нужно помнить** при использовании fifo, так как падение "читающего" процесса и последующая попытка записи в fifo без обработки SIGPIPE **приведет к падению** "пишущего" процесса. Если "читающим" процессом вы используете какой-нибудь логгер и он упадет, то и сама логгируемая программа тоже упадет, если она не умеет обрабатывать SIGPIPE!

Практический пример:

Создайте файл fifotest.py
```python
import time

print("Nothing happened yet")
with open("/tmp/myfifo", 'w') as fifo:
    print("fifo is opened for writing, waiting 10 sec before writing")
    time.sleep(10) 
    fifo.write("hello, world")
    print("write attempt performed")

print("fifo closed")
```

Выполните `mkfifo /tmp/myfifo`, затем запустите `python3 fifotest.py`

Вы увидите надпись "Nothing happened yet" и процесс приостановит выполнение. Как только вы в соседнем терминале выполните `tail -f /tmp/myfifo`, вы увидите надпись "fifo is opened for writing, waiting 10 sec before writing". На текущем этапе fifo открыт на запись, но операции записи еще не было.

И вот тут очень важное - если за эти 10 секунд вы закроете процесс tail (ctrl+c), то при попытке записи процесс python получит SIGPIPE:

```bash
Nothing happened yet
fifo opened for writing, waiting 10 sec before writing
write attempt performed
BrokenPipeError: [Errno 32] Broken pipe

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/senderman/kek.py", line 4, in <module>
    with open("/tmp/myfifo", 'w') as fifo:
         ~~~~^^^^^^^^^^^^^^^^^^^^
BrokenPipeError: [Errno 32] Broken pipe
```
