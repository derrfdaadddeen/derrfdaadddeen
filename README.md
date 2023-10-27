import tkinter as tk
from tkinter import ttk
import sqlite3

# класс главного  окна 
class Main(tk.Frame):
    def __init__(self, root):
        super().__init__(root)
        self.init_main()
        self.db = db
        self.view_records()
    
    def init_main(self):
        #инструменты
        toolbar = tk.Frame(bg='#d7d7d7', bd=2)
        toolbar.pack(side = tk.TOP, fill = tk.X)


        #создание кнопки добавления контакта
        self.add_img = tk.PhotoImage(file='./image/add.png')
        btn_add = tk.Button(toolbar,bg = '#d7d7d7', db=0,
                            image=self.add_img,
                            command=self.open_dialog)
        btn_add.pack(side=tk.LEFT)
        

        #создание кнопки редактирования контакта
        self.edit_img = tk.PhotoImage(file='./img/update.png')
        btn_eddit = tk.Button(toolbar,bg='#d7d7d7',bd=0,
                            image=self.edit_img,
                            command=self.open_edit)
        btn_eddit.pack(side=tk.LEFT)

        #создание кнопки поиска контакта
        self.search_img = tk.PhotoImage(file="./img/search.png")
        btn_search = tk.Button(toolbar,bg = '#d7d7d7',bd=0,
                               image=self.search_img,
                               command=self.open_search)
        btn_search.pack(side=tk.LEFT)

        #создание кнопки обновления
        self.refresh_img = tk.PhotoImage(file = './img/search.png')
        btn_refresh = tk.Button(toolbar, bg='#d7d7d7', bd=0,
                                image = self.refresh_img,
                                command= self.view_records)
        btn_refresh.pack(side=tk.LEFT)

        #СОЗДАНИЕ ТАБЛИЦЫ

        self.tree = ttk.Treeview(root,
                                 columns=('id','name', 'tel', 'email'),
                                 height=15,
                                 show='headings')

        # добавление параметров по столбцам
        self.tree.column('id',width=45, anchor=tk.CENTER)
        self.tree.column('name',width=300,anchor=tk.CENTER)
        self.tree.column('tel',width=150,anchor=tk.CENTER)
        self.tree.column('email',width=150,anchor=tk.CENTER)

        self.tree.heading('id',text='id')
        self.tree.heading('name',text='ФИО')  
        self.tree.heading('tel',text='Телефон')
        self.tree.heading('email',text='E-mail')
        self.tree.pack(side=tk.LEFT)  

        #полоса прокрутки 
        scroll = tk.Scrollbar(root,command=self.tree.yview)
        scroll.pack(side=tk.LEFT, fill=tk.Y)
        self.tree.configure(yscrollcommand=scroll.set)

        #метод добавления( посредник ) 
    def records(self, name, tel, email):
        self.db.insert_data(name, tel, email)
        self.view_records()


        # Метод редактировния
    def edit_record(self,name,tel,email):
        ind = self.tree.set(self.tree.selection()[0],'#1')
        self.db.cur.execute('''
            UPDATE user SET name = ?, phone = ?, email = ?
            WHERE id = 7
        ''', (name, tel, email, ind))
        self.db.conn.commit()
        self.view_records()
        

        # метод удаления записей
    def delite_records(self):
        for i in self.tree.selection():
            id = self.tree.set(i,'#1')
            set.db.cur.execute('''
                DELITE FROM users
                WHERE id = ?
            ''',(id, ))
        self.db.conn.commit()
        self.view_records()
        



    

        #метод поиска записей
    def search_records(self,name):
        [self.tree.delete(i) for i in self.tree.get_children()]
        self.db.cur.execute('SELECT * FROM users WHERE name LIKE ?', 
                            ('%'+name+'%',))
        [self.tree.insert('','end',values=i)for i in self.db.cur.fetchall()]

    #вызов дочернего окна
    def open_dialod(self):
        Child()

    # вызов редактирования
    def open_edit(self):
        Update()

    def open_search(self):
        Search()
    
    def view_record(self):
        [self.tree.delite(i)for i in self.tree.get_children()]
        self.db.cur.execute('SELECT * FROM users')
        [self.tree.insert('','end',values=i) for i in self.db.cur.fetchall()]

# Класс дочернего окна  
class Child(tk.Toplevel):
    def __init__(self):
        super().__init__(root)
        self.init_child()
        self.view = app

    def init_child(self):
        self.title('Добавление контакта')
        self.geometry('400x200')
        self.resizable(False,False)
        self.grab_set()
        self.focus_set()


    # Создание формы
        label_name = tk.Label(self,text='ФИО')
        label_name.place(x=50,y=50)
        label_tel = tk.Label(self,text='Телефон')
        label_tel.palace(x=50,y=80)   
        label_email = tk.Label(self,text='E-mail')
        label_email.place(x=50,y=110)

        self.entry_name = tk.Entry(self)
        self.entry_name.place(x=200,y=50)
        self.entry_tel = tk.Entry(self)
        self.entry_tel.place(x=200,y=80)
        self.entry_email = tk.Entry(self)
        self.entry_email.place(x=200, y=110)

        self.btn_ok = tk.Button(self, text = 'Добавить')
        self.btn_ok.bind('<Button-1>',lambda ev:
                        self.view.records(
                            self.entry_name.get(),
                            self.entry_tel.get(),
                            self.entry_email.get()
                        ))
        self.btn_ok.place(x=300,y=160)

        btn_cancel = tk.Button(self, text='Закрыть', command=self.destroy)
        btn_cancel.place(x=300,y=160)

class Update(Child):
    def __init__(self):
        super().__init__()
        self.db = db
        self.init_edit()
        self.load_data()

    def init_edit(self):
        self.title('Редактирование контакта')
        self.btn_ok.destroy()
        self.btn_ok= tk.Button(self, text='Редактировать')
        self.btn_ok.bind('<Button-1>',
                        lambda ev: self.view.edit_record(
                            self.entry_name.get(),
                            self.entry_tel.get(),
                            self.entry_email.get()))
        self.btn_ok.bind('<Button-1>', lambda ev: self.destroy(),add='+')
        self.btn_ok.place(x=300,y=160)

        # метод автозаполнения формы старыми данными
    def load_data(self):
        self.db.cur.execute('''SELECT * FORM users WHERE id = ?''',
                            self.view.tree.set(self.view.tree.selection()[0],'#1'))
        row = self.db.cur.fetchone()
        self.entry_name.insert(0,row[1])
        self.entry_tel.insert(0,row[2])
        self.entry_email.insert(0,row[3])


        # класс окна поиска 
class Search(tk.Toplevel):
    def __init__(self):
        super().__init__(root)
        self.init_search()
        self.view = app

    def init_search(self):
        self.title('Поиск контакта')
        self.geometry('300x100')
        self.resizable(False,False)
        self.grab_set()
        self.focus_set()


        label_name = tk.Label(self,text='ФИО')
        label_name.place(x=50,y=30)

        self.entry_name = tk.Entry(self)
        self.entry_name.place(x=150,y=30)
        
        self.btn_ok = tk.Button(self, text='Найти')
        self.btn_ok.bind('<Button-1>',
                         lambda ev: self.view.search_records(self.entry_name.get()))
        self.btn_ok.bind('<Button>',
                         lambda ev: self.destroy(),add='+' )
        self.btn_ok.place(x=230, y=70)

        btn_cancel = tk.Button(self, text = 'Закрыть',
                               command=self.destroy)
        btn_cancel.place(x=160,y=70)


# Класс БД
class Db:
    def __init__(self):
        self.conn = sqlite3.connect('contacts.db')
        self.cur = self.conn.cursor()
        self.cur.execute('''
                    CREATE TABLE IF NOT EXISTS users(
                         id INTEGER PRIMARY KEY,
                         name TEXT,
                         phone TEXT,
                         email TEXT
                )''')

    # метод добавления в бд
    def insert_data(self, name,tel,email):
        self.cur.execute('''
                INSERT INTO users (name,phone,email)
                VALUES (?, ?, ?)''',(name,tel,email))
        self.conn.commit()

        # Действия при запуске проги
if __name__ == '__main__':
    root = tk.Tk()
    db = Db()
    app = Main(root)
    root.title('Телефонная книга')
    root.geometry('665x400')
    root.resizable(False,False)
    root.mainloop()






<!---
derrfdaadddeen/derrfdaadddeen is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
