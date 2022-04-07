#!/usr/bin/python3
from ftplib import FTP
from datetime import datetime
from datetime import timedelta
import smtplib
from email.message import EmailMessage
from settings import *


def save_log_txt(dane):
    try:
        komunikat = datetime.now().strftime("%Y-%m-%d %H:%M:%S") + " " + str(dane)
        plik = open(FILE_PATH + 'log.txt', 'a+', encoding='utf-8') #, encoding='utf-8'
        plik.write(komunikat + "\n")
        plik.close()
        print(komunikat)
    except:
        print("Błąd zapisu log.txt!")


def save_last_chceck():
    try:
        plik = open(FILE_PATH + 'last_check.txt', 'w', encoding='utf-8')
        plik.write(str(datetime.now()))
        plik.close()
    except:
        save_log_txt("Błąd zapisu save_last_chceck.txt!")
        

def read_last_chceck():
    last_check = "2022-01-01 00:00:01.0"
    try:
        plik = open(FILE_PATH + 'last_check.txt', 'r', encoding='utf-8')
        last_check = plik.readline()
        if last_check == "":
            last_check = "2022-01-01 00:00:01.0"
        plik.close
    except:
        zapisz_err_txt("Brak lub błąd pliku read_last_check.txt!")
        
    date_format_str = '%Y-%m-%d %H:%M:%S.%f'   
    return datetime.strptime(last_check, date_format_str)    


def save_all_files(files):
    try:
        plik = open(FILE_PATH + 'all_files.txt', 'w', encoding='utf-8')
        for file in files:
            plik.write(file + "\n")
        
        plik.close()
        save_log_txt("Zapisane wszystkie pliki (" + str(len(files)) + ")")
    except:
        save_log_txt("Błąd zapisu all_files_chceck.txt!")


def read_all_files():
    try:
        plik = open(FILE_PATH + 'all_files.txt', 'r', encoding='utf-8')  
        files = plik.read().rstrip() 
        plik.close()
        return files
    except:
        save_log_txt("Błąd odczytu all_files_chceck.txt!")
        return "err"


def get_dirs_ftp(folder=""):
    contents = ftp.nlst(folder)
    folders = []
    for item in contents:
        #print(item)
        if "." not in item and "_mail" not in item and "_poprzednie" not in item:
        #if "." not in item:
            folders.append(item)
    return folders


def get_all_dirs_ftp(folder=""):
    dirs = []
    new_dirs = []

    new_dirs = get_dirs_ftp(folder)

    while len(new_dirs) > 0:
        for dir in new_dirs:
            dirs.append(dir)

        old_dirs = new_dirs[:]
        new_dirs = []
        for dir in old_dirs:
            for new_dir in get_dirs_ftp(dir):
                new_dirs.append(new_dir)

    dirs.sort()
    return dirs


def get_all_files_dir_ftp(folder=""):
    files = []

    try:
        files = ftp.mlsd(folder)
    except ftp.error_perm as resp:
        if str(resp) == "550 No files found":
            save_log_txt("Brak plików w folderze " + folder)
        else:
            raise

    
    #files.sort()
    return files


def get_new_files_dir_ftp():
    new_files = []
    all_files = []

    save_log_txt("Pobieram foldery i pliki z serwera")
    all_dirs = get_all_dirs_ftp()

    dir_prev = ""
    for dir in all_dirs:
        all_files.append("DIR: " + dir)
        # print(dir)
       
        files = get_all_files_dir_ftp(dir)
        
        for file in files:
            all_files.append("FILE: " + str(file))
            if file[1]['type'] == "file":
                # print("-->", file[0], file[1]['modify'])
                date_format_str = '%Y%m%d%H%M%S'
                if datetime.strptime(file[1]['modify'], date_format_str) > read_last_chceck() or not file[1]['unique'] in old_all_files:
                    if dir_prev != dir:
                        new_files.append("\nFOLDER: " + dir + "\n")
                    
                    dir_prev = dir
                    
                    if "." in file[0]:
                        new_files.append("PLIK: -->" + file[0] +  "\n")

        

    #new_files.sort()
    return new_files, all_files


save_log_txt("Łączę z {}".format(FTP_HOST))
ftp = FTP(FTP_HOST)
ftp.login(FTP_USER, FTP_PASSWORD)
ftp.encoding = 'utf-8'
save_log_txt("Połączony z {}".format(FTP_HOST))

old_all_files = read_all_files()

new_files, all_files = get_new_files_dir_ftp()
save_all_files(all_files)

if len(new_files)>0:
    save_log_txt("Wysyłam listę zmian na email")
    content = "Pliki dodane lub zmienione:"
    for new_file in new_files:
        save_log_txt("" + new_file)
        content = content + "" + new_file + ""

    msg = EmailMessage()
    msg.set_content(content)

    msg['Subject'] = EMAIL_SUBJECT
    msg['From'] = EMAIL_FROM
    msg['To'] = EMAIL_TO

    # Wyślij wiadomość przez serwer lokalny
    s = smtplib.SMTP(EMAIL_SMTP)
    s.send_message(msg)
    s.quit()
else:
    save_log_txt("Brak zmian")


ftp.close()
save_last_chceck()
save_log_txt("Koniec")
