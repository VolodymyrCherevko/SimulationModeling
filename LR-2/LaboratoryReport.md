## Комп'ютерні системи імітаційного моделювання
## СПм-22-3, **Черевко Володимир Геннадійович**
### Лабораторна робота №**2**. Редагування імітаційних моделей у середовищі NetLogo

<br>

### Варіант 9, модель у середовищі NetLogo:
[Shepherds](http://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/Sample%20Models/Biology/Shepherds.nlogo)

### Внесені зміни у вихідну логіку моделі, за варіантом:

**Поділити вівець та пастухів на дві організації** Пастухи повинні збирати тільки "своїх" вівець.

Поділ виконується додаванням вівцям і пастухам власного параметра *team-color*

<pre>
sheep-own 
[
  team-color                   ;; indicates which team the sheep belongs to
]
shepherds-own
[
  carried-sheep                ;; the sheep the shepherd is carrying ('nobody' if he is carrying nothing)
  team-color                   ;; indicates which team the shepherd belongs to
]
</pre>

Глобально створюємо змінні в яких будемо зберігати колір кожної команди. Зробленого для того щоб легко можна було змінити його на інший колір (в моєму випадку обрані кольори *Yellow* та *Blue* )
<pre>
globals
[
  first-team-color
  second-team-color
  ...
]

to setup
  clear-all
  set-default-shape sheep "sheep"
  set-default-shape shepherds "person"
  
  set first-team-color yellow                                     ;; set first team color
  set second-team-color blue                                      ;; set second team color

  ...

  create-sheep num-sheep
  [ 
    set team-color get-team-color                  ;; random set a team for a sheep
    set color get-sheep-color team-color           ;; according to team set sheep color
    set size 1.5                                   ;; easier to see
    setxy random-xcor random-ycor
  ]
  create-shepherds num-shepherds
  [ 
    set team-color get-team-color                 ;; random set a team for a shepherd
    set color team-color                          ;; according to team set shepherd color
    set size 1.5                                  ;; easier to see
    set carried-sheep nobody
    setxy random-xcor random-ycor
  ]

  reset-ticks
end
</pre>

Процедура *get-team-color* повертає випадковий колір команди, тож вівці і пастухи не завжди поділені рівно навпіл
<pre>
to-report get-team-color                          ;; random return team color
  ifelse (random 2 > 0)
  [ report first-team-color ]
  [ report second-team-color ]
end
</pre>

**Додати відключаєму можливість збирати "чужих" вівець**, які після потрапляння до нового стада змінюють свою приналежність:

Змінна *pick-up-any-sheep*, що встановлюється за допомогою перемикача на головному екрані моделювання, присутня в умові підбирання пастухом вівці, тож якщо значення *true* то пастухи будуть брати вівець, як зі свого стада, так і з чужого. "Чужі" вівці підібрані пастухами оновлюють значення власної властивості *team-color* на відповідне значення пастуха, який її підібрав.
<pre>
to search-for-sheep                               ;; shepherds procedure of sheep searching
  ...
  set carried-sheep one-of sheep-here with [ 
    color != red                                  ;; a sheep is not carried by someone else
    and (pick-up-any-sheep                        ;; it's allowed to take any sheep
  ...

  if (carried-sheep != nobody)
    [ ask carried-sheep
      [ 
        set color red                             ;; it makes the sheep unavailable to other shepherds while carrying
        set team-color shepherd-team-color        ;; to appropriate the sheep (for case when shepherd took someone else's sheep)
      ]
      set color team-color + 2                    ;; it indicateds that shepherd is bussy while carrying a sheep
      fd 1 ]
end
</pre>

Також, вівці на момент пернесення пастухом до власного пасовища, змінюють колір на *червоний*, замість того, щоб зникати (як це було зроблено в початковій версії), таким чином краще видно пастуха, який переносить вівцю.
<br>

### Внесені зміни у вихідну логіку моделі, на власний розсуд:

**Додано забарвлення вівці**  в залежності від кольру команди в якій вона наразі рахується.
<pre>
to-report get-sheep-color [colour]               ;; accordig to team color paint sheep
  ifelse (colour = first-team-color)
  [ report white ]
  [ report black]
end
</pre>

**Додано фіксовані координати пасовища кожної команди**
Пастухи мають висаджувати вівець тільки на території свого пасовища. В моємі прикладі центр пасовища першої команди знаходиться в точці (10; 10), другої (-10; -10)
<pre>
to-report get-pasture-cor [colour]                ;; accordig to team color return coordinates of pasture location
  ifelse (colour = first-team-color)
  [ report 10 ]
  [ report -10 ]
end
</pre>

Розмір пасовища змінюється в залежності від загальної кількость вівець, що може бути динамічно задана.
Територія пасовища поділяється на загальну територію і на зону висадки вівець, яка розташована в центрі, і офарбована в колір кожної із команд.

Всі *patches* маю властивості:
 *patch-team-color* зберігає колір команди якій належать або *nobody* якщо нікому не належить; *is-drop-off-area* взазує на те чи це місце для висадки вівець
<pre>
globals
[
  ...
  drop-off-area-radius         ;; area where shepherds have to drop off sheep
  safe-pasture-area-radius     ;; area around drop off zone where sheep can move and shepereds don't pick up them
]
patches-own
[
  patch-team-color             ;; what team the patch belongs to ('nobody' if the patch belongs to nobody)
  is-drop-off-area             ;; let shepherd to drop-off a sheep here
]

...

to setup
  ...

  set safe-pasture-area-radius ceiling ((sqrt num-sheep) / 4)
  set drop-off-area-radius ceiling (safe-pasture-area-radius / 4)

  ask patches
  [
    set pcolor green + (random-float 0.8) - 0.4    ;; varying the green just makes it look nicer
    set patch-team-color nobody                    ;; all patches belong to nobody in the begining
    set is-drop-off-area false                     ;; there are no drop off areas in the begining
    setup-pasture-area first-team-color            ;; setup pasture area for first team
    setup-pasture-area second-team-color           ;; setup pasture area for second team
  ]

  ...
end

...

to setup-pasture-area [ color-of-team ]
  let pasture-cor get-pasture-cor color-of-team
  
  if pxcor >= pasture-cor - safe-pasture-area-radius and pxcor <= pasture-cor + safe-pasture-area-radius and pycor >= pasture-cor - safe-pasture-area-radius and pycor <= pasture-cor + safe-pasture-area-radius
  [ 
    set pcolor 63.5                               ;; paint pasture area
    set patch-team-color color-of-team            ;; mark that patches belong to a team
  ]

  if (pxcor > pasture-cor - drop-off-area-radius and pxcor < pasture-cor + drop-off-area-radius
      and pycor > pasture-cor - drop-off-area-radius and pycor < pasture-cor + drop-off-area-radius)
  [ 
    set is-drop-off-area true                     ;; mark dropping off area
    set pcolor color-of-team                      ;; paint drop off area in team color
  ]
end
</pre>

**Коли пастух підібрав вівцю, він рухається чітко по направлені до свого пасовища**, замість того щоб хаотично рухатися, як під час пошуку вівці. Висаджує вівцю тільки на своєму пасовищі, і тільки в зоні для висадки вівець. Після висадки вівці, вона фарбується в колір вівець цієї команди.

На території свого пасовища пастух не підбирає "своїх" вівець, але може підбирати "чужих" вівець, якщо змінна *pick-up-any-sheep* має значення *true*.

<pre>
to search-for-sheep                               ;; shepherds procedure of sheep searching
  let shepherd-team-color team-color
  set carried-sheep one-of sheep-here with [ 
    color != red                                  ;; a sheep is not carried by someone else
    and (pick-up-any-sheep                        ;; it's allowed to take any sheep
      or team-color = shepherd-team-color)        ;; a sheep team and a shepered team are the same
    and (patch-team-color = nobody                ;; a sheep is located away from the pasture
      or patch-team-color != shepherd-team-color  ;; a sheep is located in someone else's pasture
      or team-color != shepherd-team-color) ]     ;; a sheep team and a shepherd team are different (condition reachable when 'pick-up-any-sheep' is ON and a sheep from someone else's pasture on shepherd's pasture area)
  
  if (carried-sheep != nobody)
    [ ask carried-sheep
      [ 
        set color red                             ;; it makes the sheep unavailable to other shepherds while carrying
        set team-color shepherd-team-color        ;; to appropriate the sheep (for case when shepherd took someone else's sheep)
      ]
      set color team-color + 2                    ;; it indicateds that shepherd is bussy while carrying a sheep
      fd 1 ]
end

to find-empty-spot                                ;; looking for its own pasture and area on it to drop off a sheep
  if (patch-team-color = team-color and is-drop-off-area)
  [   
    ask carried-sheep
    [ set color get-sheep-color team-color ]      ;; make the sheep accessible again
    set color team-color                          ;; make the shepherd free
    set carried-sheep nobody                      ;; make the shepherd free
  ]
  set-pasture-direction
end

to set-pasture-direction                          ;; set heading of moving to team drop off area for sheperd when he is carrying a sheep
  let pasture-cor get-pasture-cor team-color
  set heading ((atan (pasture-cor - xcor) (pasture-cor - ycor) ))
end
</pre>


**Зупинка симуляції, якщо будь-яка із команд зібрала всіх "своїх" вівець**

Розраховується відсоток зібраних вівець (зібраною вважається вівця, яка знаходиться на території свого пасовища), і якщо цей відсоток перевищує 99%, то команда яка перша зробила це, оголошується переможцем [ в консолі друкується колір команди яка перемогла].
<pre>
to go
  ifelse is-game-over
  [ stop ]                                        ;; stop simulation if any team collected all their sheep
  [ tick ]
  
  ...
end

...

to-report is-game-over                            ;; returns boolean is any of team collected more than 99% of all their sheep
  let first-team-progress calculate-team-progress first-team-color
  let second-team-progress calculate-team-progress second-team-color
  if first-team-progress < 99 and second-team-progress < 99
  [ report false ]
  
  let winner-color "Yellow"                       ;; first-team-color
  if  second-team-progress > first-team-progress
  [ set winner-color "Blue" ]                     ;; second-team-color
  
  print "** Victory **"                           ;; prints which team won
  print word winner-color " team collected all sheep"
  print word "Simulation finished at tick " ticks
  report true
end

...

to-report calculate-team-progress [ colour ]    ;; calculate how many sheep are already collected
  let collected-sheep count sheep with [ patch-team-color = team-color and team-color = colour ] ;; sheep that are already on home pasture
  let all-sheep count sheep with [ team-color = colour ] + 0.001                                 ;; all sheep of this team P.S. + 0.001 is a workaround to avoid division by zero
  report (collected-sheep / all-sheep) * 100
end
</pre>

**Монітори виводу**
Монітор з графіками показує прогрес кожної команди в графічному вигляді.

Також додано звичайні монітори, які показують кількість пастухів та вівец кожної із команд, а також відсоток зібраних вівець.


![Скріншот моделі в процесі симуляції](example-model.png)
<br>

## Обчислювальні експерименти

### 1. Вплив швикості вівець на час їх збирання
Досліджується залежність часу збирання вівець від  швидкості їх пересування.
Експерименти проводяться при швидкостях 0-0.05 у.о. з кроком 0.01 , усього 5 симуляцій.  
Інші керуючі параметри мають значення за замовчуванням:
- **num-shepherds 1**: 50
- **num-shepherds 2**: 100
- **num-sheep**: 250

<table>
<thead>
<tr><th>Sheep speed</th><th>Collecting time, [50 sh]</th><th>Collecting time, [100 sh]</th></tr>
</thead>
<tbody>
<tr><td>0</td><td>461</td><td>265</td></tr>
<tr><td>0.01</td><td>551</td><td>293</td></tr>
<tr><td>0.02</td><td>655</td><td>320</td></tr>
<tr><td>0.03</td><td>48 072</td><td>425</td></tr>
<tr><td>0.04</td><td>endless</td><td>2458</td></tr>
<tr><td>0.05</td><td>endless</td><td>endless</td></tr>
</tbody>
</table>

![Залежність часу збирання вівець від  швидкості їх пересування](fig1.png)

Графік наочно показує, що зі збільшенням швидкості пересування вівці, збільшується час їх збирання до пасовища. А при досягнені швидкості вівці більше 0.04 у.о. час прагне до нескінченості. Навіть збільшення кількості пастухів вдвічі не може суттєво вплинути на час.