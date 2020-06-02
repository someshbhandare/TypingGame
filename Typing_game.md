
from tkinter import *
from tkinter import messagebox
from threading import Timer
import random
import sqlite3

correct_words = 0
wrong_words = 0
total_words = 0
ended=False

def back(win,num):
    win.destroy()
    win.quit()

def quite_game():
    Window.quit()

def get_speed(correct, wrong, total):
    accuracy = round(((correct / total) * 100))
    speed = float(correct / 2)
    stats = (correct, wrong, accuracy, speed)
    return stats


def check_word(reference_word, given_word,lable3,Answer):

    if reference_word == given_word:
        print("Right")
        global correct_words
        correct_words += 1

    else:
        global wrong_words
        print("Wrong")
        wrong_words += 1
    next_word(lable3,Answer)

def next_word(label3,Answer):
    global total_words
    label3=label3
    word_list = get_words()
    random_word = random.choice(word_list)
    total_words += 1
    label3.config(text=random_word)
    Answer.delete(0,END)
    Answer.bind('<Return>', lambda event :check_word(str(random_word),str(Answer.get()),label3,Answer))

def get_words():
    words = []
    with open('words.txt', 'r') as f:
        words = f.read().split("\n")
    return words

def instruction():
    ins=Toplevel()
    ins.title(" Instruction ")
    ins.geometry("700x400")
    lable1=Label(ins,text=" 1) A word will come up on the screen and you'll have to type the",font=("Arieal",16,"bold"))
    lable1.place(x=10,y=20)
    lable2=Label(ins,text="      exact same word and hit enter.",font=("Arieal",16,"bold"))
    lable2.place(x=10,y=60)
    lable3=Label(ins,text=" 2) You'll have 10 seconds to type in as many words as you can.",font=("Arieal",16,"bold"))
    lable3.place(x=20,y=100)
    b_go_back=Button(ins,text="GO Back",font=("Arieal",20,"bold"),command=lambda :back(ins,1))
    b_go_back.place(x=60,y=160)
    ins.mainloop()

def view_record():
    con = sqlite3.connect("Typing_Game")
    c = con.cursor()
    rec = Toplevel()
    rec.title(" Records ")
    rec.geometry("700x400")
    c.execute(" SELECT * FROM Records")
    datas=c.fetchmany(5)
    print(datas)
    con.commit()
    con.close()
    lable1 = Label(rec,text=" Top 5 Higest Record ",font=("Arieal", 16, "bold"))
    lable1.pack()
    lable2 = Label(rec, text=" Correct" + "  " + "Wrong" + "  " + " Accurecy " + "  " + "speed", font=("Arieal", 16, "bold"))
    lable2.pack()
    for data in datas:
        lable2 = Label(rec, text=str(data[0]) + "\t  "+ str(data[1])+" \t "+str(data[2])+" \t "+str(data[3]), font=("Arieal", 16, "bold"))
        lable2.pack()
    b_go_back = Button(rec, text="GO Back", font=("Arieal", 18, "bold"), command=lambda: back(rec, 1))
    b_go_back.place(x=500, y=300)
    rec.mainloop()

def end_game(game):
    con = sqlite3.connect("Typing_Game")
    c = con.cursor()
    global ended
    global correct_words,wrong_words,total_words
    ended = True
    for i in range(1, 3):
        game.destroy()
    status=get_speed(correct_words,wrong_words,total_words)
    print(status)
    messagebox.showinfo("Game Finished \n","\n Correct Word:- "+str(status[0])+"\n Wrong Word :-"+str(status[1])+"\n Accuracy :- "+str(status[2])+"%"+"\n Speed :- "+str(status[3])+"wps")
    c.execute("INSERT INTO Records(correct,wrong,accurcy,speed) values ('"+str(status[0])+"' ,'"+str(status[1])+" ',' "+str(status[2])+" ',' "+str(status[3])+" ' )")
    con.commit()
    con.close()

def play():
    game = Toplevel()
    game.title(" Play ")
    game.geometry("700x400")
    lable1 = Label(game, text="Game Start in ", font=("Arieal", 16, "bold"))
    lable1.pack()
    lable2 = Label(game, text=" Word :-", font=("Arieal", 16, "bold"))
    lable2.place(x=150,y=110)
    word_list = get_words()
    global correct_words,wrong_words,total_words
    correct_words = 0
    wrong_words = 0
    total_words = 0
    word_list = get_words()
    # Call end_game() after 120 seconds
    t = Timer(10, lambda :end_game(game))
    t.start()
    if ended:
        exit()
    random_word = random.choice(word_list)
    total_words += 1
    lable3 = Label(game, text=random_word, font=("Arieal", 16, "bold"))
    lable3.place(x=300, y=110)
    Answer=Entry(game,font=("Arieal",16,"bold"))
    Answer.place(x=200,y=200)
    Answer.bind('<Return>', lambda event :check_word(str(random_word),str(Answer.get()),lable3,Answer))
    b_go_back = Button(game, text="GO Back", font=("Arieal", 18, "bold"), command=lambda: back(game, 1))
    b_go_back.place(x=500, y=300)
    game.mainloop()

if __name__ == '__main__':
    Window=Tk()
    Window.title("Typing Speed ")
    Window.geometry("700x400")

    title=Label(Window,text="Typing Speed ",font=("Arieal",22,"bold"))
    title.place(x=250,y=10)

    welcome=Label(Window,text=" Welcome ",font=("Arieal",18,"bold"))
    welcome.place(x=300,y=115)

    b_play=Button(Window,text="Play",font=("Arieal",18,"bold"),borderwidth=5,command=play)
    b_play.place(x=60,y=200,height=100,width=130)

    b_view=Button(Window,text="Records",font=("Arieal",18,"bold"),borderwidth=5,command=view_record)
    b_view.place(x=210,y=200,height=100,width=130)

    b_ins=Button(Window,text="Help",font=("Arieal",18,"bold"),borderwidth=5,command=instruction)
    b_ins.place(x=350,y=200,height=100,width=130)

    b_quite=Button(Window,text="Quite",font=("Arieal",18,"bold"),borderwidth=5,command=quite_game)
    b_quite.place(x=500,y=200,height=100,width=130)

    Window.mainloop()
