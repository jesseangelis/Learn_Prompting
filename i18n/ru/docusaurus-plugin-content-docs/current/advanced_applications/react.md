---
sidebar_position: 3
---

# 🟡 LLMs которые рассуждают и действуют

ReAct(@yao2022react)(reason, act) - это парадигма, позволяющая языковым моделям решать сложные задачи с помощью рассуждений на естественном языке. ReAct предназначен для задач, в которых LLM разрешено выполнять определенные действия. Например, как в системе MRKL, LLM может иметь возможность взаимодействовать с внешними API для получения информации. Когда ЛЛМ задают вопрос, он может выбрать действие для получения информации, а затем ответить на вопрос на основе полученной информации.

Системы ReAct можно рассматривать как системы MRKL, с дополнительной способностью **рассуждать о** действиях, которые они могут выполнять.

Рассмотрите следующее изображение. Вопрос в верхнем поле взят из HotPotQA (@yang2018hotpotqa), набора данных для ответов на вопросы, требующих сложных рассуждений. ReAct способен ответить на вопрос, сначала обдумав его (Thought 1), а затем выполнив действие (Act 1) по отправке запроса в Google. Затем он получает наблюдение (Obs 1) и продолжает цикл "мысль, действие, наблюдение".
пока не придет к заключению (действие 3). 


import react_qa from '@site/docs/assets/advanced/react_qa.webp';

<div style={{textAlign: 'center'}}>
  <img src={react_qa} style={{width: "500px"}} />
</div>

<div style={{textAlign: 'center'}}>
Система ReAct (Yao и др.)
</div>


Читатели, знакомые с обучением подкрепления, могут распознать этот процесс как похожий на классический цикл RL: состояние, действие, вознаграждение, состояние, .... ReAct предлагает некоторую формализацию этого процесса в своей статье. 


## Результаты

В экспериментах с ReAct компания Google использовала LLM PaLM(@chowdhery2022palm). 
Сравнение со стандартным промтингом (только вопрос), CoT и другими конфигурациями показало, что производительность ReAct многообещающа для сложных задач рассуждения. Google также проводит исследования на наборе данных FEVER (@thorne2018fever), который охватывает извлечение и проверку фактов.  

import react_performance from '@site/docs/assets/advanced/react_performance.webp';

<div style={{textAlign: 'center'}}>
  <img src={react_performance} style={{width: "500px"}} />
</div>

<div style={{textAlign: 'center'}}>
Результаты ReAct (Yao и др.)
</div>

