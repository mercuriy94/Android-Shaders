# Shaders in Android

### Введение

Время от времени Android разработчики пишут кастомные вьюхи. В которых реализуют интересные задумки интерфейса, чтобы привлечь внимание пользователей. Не сомнено, в текущий момент мы отдаем большое предпочтение приложениям которые имеют красивый и отзывчивый интерфейс. И скорее всего каждый разработчик имеет желание добавить в интерфейс изюминку, за котороую пользователи будут любить ваше приложение и хотеть в нем проводить время. И это вполне очевидно. В данной серии статей я постараюсь раскрыть прицнип работы известных шейдеров, что я надеюсь вдохновит вас разрабатывать красивые и плавные  элементы представления.
 
### Shaders
   Чтобы уловить смысл шейдеров, предлагаю воспользоваться термином который употребил Виталий Непочатов в своей серии уроков -  "Рисовать рисунком".  Думаю это вполне описывает данный процесс. На протяжении данных статей я постараюсь показать примеры использования шейдеров в разработке, которые вы сможете интегрировать в свои проекты.
  
  Популярных шедеров на самом неделе не так много, но все они по своему интересны. SDK Android из коробки даёт нам следюущий набор шейдеров:
  
  - [LinearGradient ](https://developer.android.com/reference/android/graphics/LinearGradient) ([Ссылка ](https://android.googlesource.com/platform/frameworks/base/+/master/graphics/java/android/graphics/LinearGradient.java) на исходник);
  - [SweepGradient ](https://developer.android.com/reference/android/graphics/SweepGradient) ([Ссылка ](https://android.googlesource.com/platform/frameworks/base/+/master/graphics/java/android/graphics/SweepGradient.java) на исходник);
  - [RadialGradient ](https://developer.android.com/reference/android/graphics/RadialGradient.html)  ([Ссылка ](https://android.googlesource.com/platform/frameworks/base/+/master/graphics/java/android/graphics/RadialGradient.java) на исходник);
  - [BitmapShader ](https://developer.android.com/reference/android/graphics/BitmapShader) ([Ссылка ](https://android.googlesource.com/platform/frameworks/base/+/master/graphics/java/android/graphics/BitmapShader.java) на исходник);
  - [ComposeShader ](https://developer.android.com/reference/android/graphics/ComposeShader) ([Ссылка ](https://android.googlesource.com/platform/frameworks/base/+/master/graphics/java/android/graphics/ComposeShader.java) на исходник);

 Все перечисленные классы шейдеров унаследованны от базового класса  [Shader](https://android.googlesource.com/platform/frameworks/base/+/master/graphics/java/android/graphics/Shader.java). В котором отведенно поле для матрицы (Matrix) преобразования шейдар (Как использовать матрицу можно почитать в данной [серии ](https://ru.smedialink.com/razrabotka/matritsy-v-android-pervaya-chast/) статей). Пречисление TileMode, которое определяет поведение для заполнения свободного пространства, нам доступны следующие режимы (Более подробно мы поговорим о них позже режимах позже.): 
 
 - CLAMP ();
 - REPEAT ();
 - MIRROR ();

А также имеется поле, для сохарнения указателя на экземпляр нативного класса (mNativeInstance).
Ну и пожалуй в данном классе мы ничего интересного больше не встретим. Итак, вернемся к исходникам самих шейдеров. Я в качестве примера рассмотрю LinearGradient. Имеется два конструктора и парочка нативных методов: nativeCreate1 и nativeCreate2. Но до сих пор нам не ясно кто и как будет рисовать наши рисунки. Для этого нам нужно немного погрузиться в нативные классы Android. 
 Первой остановкой будет [Shader.cpp](https://android.googlesource.com/platform/frameworks/base/+/master/core/jni/android/graphics/Shader.cpp). Данный класс являтся своего рода фабрикой и внизу файла содржится метод регистрирующий нативные методы. И составляет труда найти реализацию двух методов для создание экземпляра LinearGradient. Обратите внимание, что данные методы имеют следующие характеристики:
  -  Возвращают примитивный тип jlong, который является указателем на экземпляр нативного класса SkShader, который присваивается полю mNativeInstance в рассмотренном ранее нами класса Shader.java. 
 - Параметр JNIEnv* env - указатель на виртуальную машину;
 - Параметр jobject o - ссылка на объект из которого идет данный вызов метода, в нашем случае экземлпяр java класса. 
 Остальные параметры совпадают с параметрами из точки вызова данных методов.
 
 Если вы как и я немного заинтересовались содержимым методов, то скорее всего тоже заметили, что для создания линейного градиента используется класс [SkGradientShader](https://github.com/google/skia/blob/master/include/effects/SkGradientShader.h), вызывая его методы MakeLinear. Вот тут я для себя сделал небольшое открытие, которое вам я думаю тоже понравится.
Как оказывается, для рисования двумерной граффики в Android используется библиотека [Skia](https://skia.org/).  
Это библиотека с открытым исходным кодом, для рисование 2D графики. Нам предлагаются API - интерфейсы, для работы на различных платформах. Как утверждают разработчики библиотеки, она используетя не только в Android, но и в Google Chrome, Chrome OS, Mozilla Firefox и Firefox OS. 
Первым делом я начал просматривать справочник API. Но к сожалению не нашел отдельной страницы посвященной шейдерам, но я наткнулся на их описание когда просматривал обозревательную статью [SkPaint Overview](https://skia.org/user/api/skpaint_overview). 
Тут нам показывают небольшие сэмплы с применением нескольких шейдеров. Обратите внимание, что В природе шейдеров гораздо больше, нежеле чем нам предлагается в Android SDK, вот несколько из них:

  - Two-Point Conical Gradient Shader;
  
  ![](https://fiddle.skia.org/i/@skpaint_2pt_raster.png) 
  
  - Fractal Perlin Noise Shader;

  ![](https://fiddle.skia.org/i/@skpaint_perlin_raster.png) 
  
  - Turbulence Perlin Noise Shader;

   ![](https://fiddle.skia.org/i/@skpaint_turb_raster.png) 
   
  Предлагаю подвести небольшой итог вводной части.
Все доступные шейдеры унаследованны от базовго классы Shader.java. Который является прослойкой между клиентским кодом и нативными вызовами. И в основе всего лежит библиотке Skio. Теперь мы знаем, что разработчики в качетсве инструментария дали нам ограниченное кол-во шейдеров. Чем вызванно данное решение, я могу только догадываться. Далее, давайте перейдем к рассмотрению самих шейдеров.
 
### LinearGradient

  Первый градиент, который мы будем рассматривать, это линейный градиент. Он представляет собой смешение нескольких выбранных цветов, направление которых определяют две точки, таким образом вводят понятие градиентной линии. И выглядит это все примерно так:
  
  ![](https://github.com/mercuriy94/Shaders-in-Android/blob/master/Resources/linear_gradient_1.png?raw=true) 

Рассмотреть как реализовать такой градиент в Android.  

~~~ kotlin
  ...

import android.content.Context
import android.graphics.*
import android.support.v7.app.AppCompatActivity
import android.os.Bundle
import android.view.View


class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(LinearShaderView(this))
    }

    private class LinearShaderView(context: Context) : View(context) {

        override fun onDraw(canvas: Canvas) {
            super.onDraw(canvas)

            val linearGradientPaint = Paint(Paint.ANTI_ALIAS_FLAG)

            val linearGradientShader = LinearGradient(
                0f,
                0f,
                canvas.width.toFloat(),
                0f,
                intArrayOf(Color.RED, Color.GREEN, Color.BLUE),
                floatArrayOf(0f, 0.5f, 1f),
                Shader.TileMode.CLAMP
            )

            linearGradientPaint.shader = linearGradientShader
            canvas.drawRect(
                0f,
                0f,
                canvas.width.toFloat(),
                canvas.height.toFloat(),
                linearGradientPaint
            )
        }
    }

}

...

~~~

Алгоритм рисования шейдером достаточно прост. Создаем экземпляр объекта необходимого шейдера, настраиваем его должным образом и передаем его объекту Paint. Который в свою очередь используется для рисования различных фигур, в данном случае мы рисовали прямоугольник. 
  Для того, чтобы создать линейный градиент нам доступны два конструктора, первый выглядит так:

~~~ java

    public LinearGradient(float x0,
                          float y0,
                          float x1,
                          float y1,
                          @NonNull @ColorInt int colors[],
                          @Nullable float positions[],
                          @NonNull TileMode tile)

~~~

   Где:
   x0,y0 - координаты точки начала градиента;
   x1,y1 - координаты точки конца градиента;
   colors[] - массив цветов входящих в данный градиент;
   positions[] - массив, определяющий позиции каждого цвета по направляющей данного градиента. Данный параметр может не устанавливаться в значние null, в таком случае цвета будут распределенны равномерно.
   tile - режим шейдера.

Отметьте, что массивы цветов и позиции клонируются, тем самым обесепечивается безопасность переданных данных. А вы после вызова конструктора линейного градиента можете выполнять различные действия над этими массивами не переживая за шейдер, так как на нем это никак не отобразится.

Второй констурктор немного проще:

~~~ java

   public LinearGradient(float x0,
                          float y0,
                          float x1,
                          float y1,
                          @ColorInt int color0,
                          @ColorInt int color1,
                          @NonNull Shader.TileMode tile)

~~~

  Где:
  color0 - Первый выбранный цвет;
  color1 - Первый выбранный цвет;
  
  Остальные поля совпадаются по смыслу с параметрами первого конcтруктора. Позиции цветов соответствуют началу и концу градиентной линии. 
  Подозреваю, что нужно более подробно раскрыть смысл понятия градиентной линии. Как было сказанно ранее - направление данной линии определяют две точки, соответствующие началу и концу градиента. 
  
Представим, что у нас имеется полотно размером 800 х 600 пикселей. 

   ![](https://github.com/mercuriy94/Shaders-in-Android/blob/master/Resources/base_canvas_800x600.png?raw=true) 
   
Если при создании градиента установим направление линии следующими точками: P1 (0, 0); P2 (0: 800).
То направление градиентной линии будет направлено слевого-верхнего края в правый верхний край, и выглядит следующим образом.

   ![](https://github.com/mercuriy94/Shaders-in-Android/blob/master/Resources/linear_gradient_2_800x600.png?raw=true) 

~~~ kotlin
  ...

  override fun onDraw(canvas: Canvas) {
            super.onDraw(canvas)

            val linearGradientPaint = Paint(Paint.ANTI_ALIAS_FLAG)

            val p1 = PointF(0f,0f)
            val p2 = PointF(800f,0f)

            val linearGradientShader = LinearGradient(
                p1.x,
                p1.y,
                p2.x,
                p2.y,
                intArrayOf(Color.RED, Color.GREEN, Color.BLUE),
                floatArrayOf(0f, 0.5f, 1f),
                Shader.TileMode.CLAMP
            )

            linearGradientPaint.shader = linearGradientShader

            canvas.drawRect(
                0f,
                0f,
                800f,
                600f,
                linearGradientPaint
            )

        }
...

~~~

Например, чтобы установить направление градиентной линии с нижнего левого края в верхний правый, то наверно вы уже догодались, что нам надо изменить координаты только первой точки в значения:

~~~ kotlin
  ...
  
     val p1 = PointF(0f,600f)
     val p2 = PointF(800f,0f)
  ...

~~~

![](https://github.com/mercuriy94/Shaders-in-Android/blob/master/Resources/linear_gradient_3_800x600.png?raw=true) 
 
 Ну и чтобы закрепить, напрами градиентную линию с нижнего правого края в правй верхний край.
 Значения точек получаются слeдующие:

~~~ kotlin
  ...
  
     val p1 = PointF(800f,600f)
     val p2 = PointF(0f,0f)
  ...

~~~

![](https://github.com/mercuriy94/Shaders-in-Android/blob/master/Resources/linear_gradient_4_800x600.png?raw=true) 

Надеюсь, у меня получилось до нести до вас смысл градиентной линии. К слову мы могли добиться такого же успеха используя матрицу преобразования. Например, чтобы получить  результат как в последнем примере, нам необходимо выполнить поворот на 225 градусов с опорной точкой рпсположенной в центре градиента (подробнее об опорной точке и как она влияет на результат можете почитать в данной  [статье](https://ru.smedialink.com/razrabotka/matritsy-v-android-vtoraya-chast/)).

~~~ kotlin
  ...
  
        override fun onDraw(canvas: Canvas) {
            super.onDraw(canvas)

            val linearGradientPaint = Paint(Paint.ANTI_ALIAS_FLAG)

            val p1 = PointF(0f, 0f)
            val p2 = PointF(800f, 0f)

            val degrees = 225f
            val rotatePoint = PointF(400f, 300f)

            val linearGradientShader = LinearGradient(
                p1.x,
                p1.y,
                p2.x,
                p2.y,
                intArrayOf(Color.RED, Color.GREEN, Color.BLUE),
                floatArrayOf(0f, 0.5f, 1f),
                Shader.TileMode.CLAMP
            ).apply {
                setLocalMatrix(Matrix().apply {
                    postRotate(degrees, rotatePoint.x, rotatePoint.y)
                })
            }

            linearGradientPaint.shader = linearGradientShader

            canvas.drawRect(
                0f,
                0f,
                800f,
                600f,
                linearGradientPaint
            )
        }
  ...

~~~

Концептуально это сложнее. Но задачи на практике задачи бывают разные, а это значит, что и этому способу найдется применение.

До данного момента, мы с вами рассматривали примеры на которые не оказывает влияние поле Shader.TileMode.

Как было сказанно раннее, Tilemode определяет поведение для заполнения свобдной области на холсте, которая выходит за периметр шейдера. Может не совсем понятно, но тут лучше один раз увидеть, чем 100 раз услышать.

Прежде, давайте подготовим заготовочку, которая просто будет рисовать градиент с направлением линии слева направо.

~~~ kotlin

 override fun onDraw(canvas: Canvas) {
            super.onDraw(canvas)

            val linearGradientPaint = Paint(Paint.ANTI_ALIAS_FLAG)

            val p1 = PointF(0f, 0f)
            val p2 = PointF(270f, 0f)

            val linearGradientShader = LinearGradient(
                p1.x,
                p1.y,
                p2.x,
                p2.y,
                intArrayOf(Color.RED, Color.BLUE, Color.GREEN),
                floatArrayOf(0f, 0.5f, 1f),
                Shader.TileMode.REPEAT
            )

            linearGradientPaint.shader = linearGradientShader

            canvas.drawRect(
                0f,
                0f,
                270f,
                270f,
                linearGradientPaint
            )
        }

~~~

![](https://github.com/mercuriy94/Shaders-in-Android/blob/master/Resources/screenshot_tilemode_1.png?raw=true)

### TileMode.Repeat

Теперь чтобы, увидеть влияние TileMode будем рисовать квадрат со сторонами ширины экрана.

~~~ kotlin

 override fun onDraw(canvas: Canvas) {
            super.onDraw(canvas)
    
      ...
            canvas.drawRect(
                0f,
                0f,
                canvas.width.toFloat(),
                canvas.width.toFloat(),
                linearGradientPaint
            )
            
        }

~~~

![](https://github.com/mercuriy94/Shaders-in-Android/blob/master/Resources/screenshot_tilemode_2.png?raw=true)


Для удобства на следующем изображении и следующих изображений я буду чертить линии разделяющие квадраты, где левый верхний квадрат будет являться исходным,, а все остальные результатом поля TileMode.
  
![](https://github.com/mercuriy94/Shaders-in-Android/blob/master/Resources/screenshot_tilemode_3.png?raw=true)

Как только линия градиента достигает конечной точки, то она градиент начинает повторятся. 
Данный случай на самом деле довольно простой, более интересно посмотреть как заполнится свободное пространство , если направить линию граиенда под наклоном , например с верхнего левого края в нижний правый край. 

![](https://github.com/mercuriy94/Shaders-in-Android/blob/master/Resources/screenshot_tilemode_4.png?raw=true)

### TileMode.Mirror

### TileMode.Clamp
  
### SweepGradient

### RadialGradient

### BitmapShader

### ComposeShader

### Итоги