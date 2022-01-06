# События жизненного цикла

Приложение Nest, как и каждый элемент приложения, имеет жизненный цикл, управляемый Nest. Nest предоставляет 
**хуки жизненного цикла**, которые обеспечивают видимость ключевых событий жизненного цикла и возможность действовать 
(запускать зарегистрированный код в вашем `модуле`, `инжектере` или `контроллере`), когда они происходят.

## Последовательность жизненного цикла

На следующей диаграмме показана последовательность ключевых событий жизненного цикла приложения, начиная с момента 
загрузки приложения и заканчивая завершением процесса узла. Мы можем разделить общий жизненный цикл на три фазы: 
**инициализация**, **запуск** и **завершение**. Используя этот жизненный цикл, вы можете планировать соответствующую 
инициализацию модулей и служб, управлять активными соединениями и изящно завершать работу приложения, когда оно получает 
сигнал о завершении.

<img src="/lifecycle-events.png" />
