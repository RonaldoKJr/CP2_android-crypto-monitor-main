# CP2_android-crypto-monitor-main

O projeto Crypto Monitor é uma aplicação em Kotlin, que usa uma API para podermos observar o valor do Bitcoin em tempo real.

## Explicação do código:

### AndroidManifest.xml

```python
    <!--Instancia a permissão do uso da internet-->
    
    <uses-permission android:name="android.permission.INTERNET" />

    <application
    
      <!--Possibilita o Backup do projeto-->
      
      android:allowBackup="true"
      android:dataExtractionRules="@xml/data_extraction_rules"
      android:fullBackupContent="@xml/backup_rules"

      <!--Define ícone, nome (label), tema e a orientação da aplicação-->
      <!--supportsRtl="true", define que a orientação é da direita para a esquerda-->
      
      android:icon="@mipmap/ic_launcher"
      android:label="@string/app_name"
      android:roundIcon="@mipmap/ic_launcher_round"
      android:supportsRtl="true"
      android:theme="@style/Theme.Kotlinandroidcryptomonitor"
      tools:targetApi="31">
    
        <activity

            <!--android:name=".MainActivity", é um atributo utilizado para "extender" (fazer o extends) da classe MainActivity-->
            <!--android:exported="true", indica que o componente pode ser iniciado por componentes de outra aplicação-->
            
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.AppCompat.Light.NoActionBar">
            
            <intent-filter>
            
                <!--Configura a MainActivity como tela principal.-->
                
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
```

### TicketResponse.kt

```python
    <!--Cria um "ticker da classe Ticker"-->
    <!--O ticker receberá os valores direto da API-->
    
    class TickerResponse(
      val ticker: Ticker
    )
    
    <!--Instancia a classe Ticker como detentora dos seguintes atributos-->
    
    class Ticker(
      val high: String,
      val low: String,
      val vol: String,
      val last: String,
      val buy: String,
      val sell: String,
      val date: Long
    )
```

### MercadoBitcoinServiceFctory.kt

```python
    <!--Configura o RetroFit para chamar a API-->
    
    fun create(): MercadoBitcoinService {
        val retrofit = Retrofit.Builder()
            .baseUrl("https://www.mercadobitcoin.net/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            return retrofit.create(MercadoBitcoinService::class.java)
  }
```

### MercadoBitcoinService.kt

```python
    <!--Define o endpoint da API, transformando a API HTTP em uma interface utilizando RetroFit-->
    
    interface MercadoBitcoinService {  
    @GET("api/BTC/ticker/")  
    suspend fun getTicker(): Response<TickerResponse>  
    }
```

### Color.kt
```python
    <!--Instancia algumas cores personalizdas-->

    val Purple80 = Color(0xFFD0BCFF)
    val PurpleGrey80 = Color(0xFFCCC2DC)
    val Pink80 = Color(0xFFEFB8C8)
    val Purple40 = Color(0xFF6650a4)
    val PurpleGrey40 = Color(0xFF625b71)
    val Pink40 = Color(0xFF7D5260)
```

### Theme.kt
```python
    <!--Define os esquemas de cor dos temas -->

    private val DarkColorScheme = darkColorScheme(
      primary = Purple80,
      secondary = PurpleGrey80,
      tertiary = Pink80
    )

    private val LightColorScheme = lightColorScheme(
      primary = Purple40,
      secondary = PurpleGrey40,
      tertiary = Pink40
      )

    <!--Composable é aplicado para indicar que uma função pode ser usada como composição para descrever uma transformação de dados para uma árvore ou hierarquia-->
    
    @Composable
    fun KotlinandroidcryptomonitorTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
    ) {
    
    <!--Define quando será utilizado cada tema-->
    
      val colorScheme = when {
          dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
              val context = LocalContext.current
              if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
          }
          darkTheme -> DarkColorScheme
          else -> LightColorScheme
        }
        val view = LocalView.current
        if (!view.isInEditMode) {
            SideEffect {
                val window = (view.context as Activity).window
                window.statusBarColor = colorScheme.primary.toArgb()
                WindowCompat.getInsetsController(window, view).isAppearanceLightStatusBars = darkTheme
            }
        }
        
    <!--Integra o uso da tipografia e esquemas de cor e ao tema-->
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
```

### Type.kt

```python
<!--Define a tipografia, ou seja, coisas relacionadas ao texto, como tamanho, espaçamento, tipo e "intensidade"-->

      val Typography = Typography(
      bodyLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
      )
    )
```

### MainActivity.kt

```python
<!--Cria a activity e verifica chamas os .xml responsáveis pelo layout-->

    class MainActivity : AppCompatActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)
        
      <!--Configura a toolbar-->
      
      val toolbarMain: Toolbar = findViewById(R.id.toolbar_main)
      configureToolbar(toolbarMain)
  
      <!--Configura o botão Refresh-->
      
      val btnRefresh: Button = findViewById(R.id.btn_refresh)
      
      <!--Chama a função "makeRestCall()" quando clickar no botão-->
      
      btnRefresh.setOnClickListener {
          makeRestCall()
       }
     }
      <!--Customiza a toolbar-->
      
      private fun configureToolbar(toolbar: Toolbar) {
      setSupportActionBar(toolbar)
      toolbar.setTitleTextColor(getColor(R.color.white))
      supportActionBar?.setTitle(getText(R.string.app_title))
      supportActionBar?.setBackgroundDrawable(getDrawable(R.color.primary))
      }
      
      <!--Cria a função "makeRestCall()"-->
      
      private fun makeRestCall() {
          CoroutineScope(Dispatchers.Main).launch {
              try {
              
              <!--Cria uma instância do serviço recebido pelo "MercadoBitcoinServiceFactory()"-->
              
                  val service = MercadoBitcoinServiceFactory().create()
                  
                  <!--Verifica a resposta recebido pelo "getTicker()"-->
                  
                  val response = service.getTicker()
                  if (response.isSuccessful) {
                  
                  <!--Se a resposta for bem-sucedida, então ocorreram os seguintes passos-->
                  
                       val tickerResponse = response.body()
                       
                       <!--Atualiza os componentes TextView-->
                       
                       val lblValue: TextView = findViewById(R.id.lbl_value)
                       val lblDate: TextView = findViewById(R.id.lbl_date)
                       
                       <!--Verifica o valor recebido-->
                       
                       val lastValue = tickerResponse?.ticker?.last?.toDoubleOrNull()
                       if (lastValue != null) {
                       
                       <!--Verifica a moeda utilizada e a atualiza (uma vez que a moeda utilizada para a verificação padrão de preço é o dólar)-->
                       
                       val numberFormat = NumberFormat.getCurrencyInstance(Locale("pt", "BR"))
                       lblValue.text = numberFormat.format(lastValue)
                   }
                   
                  <!--Verifica o formato de data e o atualiza-->
                  
                  val date = tickerResponse?.ticker?.date?.let { Date(it * 1000L) }
                  val sdf = SimpleDateFormat("dd/MM/yyyy HH:mm:ss", Locale.getDefault())
                  lblDate.text = sdf.format(date)
                  
                  <!--Caso a resposta do getTicker() não seja bem-sucedida, então ocorrem os seguintes passos-->
                  
            } else {
            
                <!--Verifica o erro e o imprime-->
                
                val errorMessage = when (response.code()) {
                   400 -> "Bad Request"
                   401 -> "Unauthorized"
                   403 -> "Forbidden"
                   404 -> "Not Found"
                   else -> "Unknown error"
                }
              Toast.makeText(this@MainActivity, errorMessage, Toast.LENGTH_LONG).show()
            }
          } catch (e: Exception) {
          
              <!--Trata o erro de falha na chamada-->
              
              Toast.makeText(this@MainActivity, "Falha na chamada: ${e.message}", Toast.LENGTH_LONG).show()
          }
        }
      }
    }
```

activity_main.xml, component_button_refresh.xml, component_quote_information.xml e component_toolbar_main.xml definem altura, largura, posição na tela, tamanho e estilo de textos, cor de background e enter outros elementos, dentro do seu próprio layout.

<!--activity_main é relacionado a posição dos itens na tela-->

### activity_main.xml

```python
        <include
        android:id="@+id/component_toolbar"
        layout="@layout/component_toolbar_main"
        android:layout_width="match_parent"
        android:layout_height="75dp"
        android:layout_weight="0" />
        
        <include
        android:id="@+id/component_quote_information"
        layout="@layout/component_quote_information"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

```
<!--component_button_refresh é relacionado ao botão utilizado para atualizar as informações na tela-->

### component_button_refresh.xml

```python
        <Button
        android:id="@+id/btn_refresh"
        android:layout_width="120dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:background="@drawable/shape_button_refresh"
        
        <!--O texto utilizado está contido na variável "label_refresh", onde seu texto está mockado-->
        
        android:text="@string/label_refresh"
        
        <!--A cor utilizada está armazenada numa variável instânciada dentro de colors.xml-->
        
        android:textColor="@color/white" />
```

<!--component_quote_information é relacionado ao texto em display na tela-->

### component_quote_information.xml

```python
        <!--O texto utilizado está contido na variável "label_rate"-->
        
        <TextView
        android:id="@+id/lbl_rate"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/label_rate"
        android:textSize="20sp" />
        
        <!--O texto utilizado está contido na variável "label_value"-->
        
        <TextView
        android:id="@+id/lbl_value"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/label_value"
        android:textSize="32sp"
        android:textStyle="bold" />
        
        <!--O texto utilizado está contido na variável "label_date"-->
        
        <TextView
        android:id="@+id/lbl_date"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/label_date" />
        
        <!--Aqui o botão criado em component_button_refresh está sendo instanciado para integrar esta tela de layout-->
        
        <include
        layout="@layout/component_button_refresh"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
```

<!--component_toolbar_main é relacionado ao local da interface onde o cabeçalho será mostrado-->

### component_toolbar_main.xml

```python
        <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar_main"
        android:layout_width="match_parent"
        android:layout_height="70dp"
        
        <!--A cor utilizada está armazenada numa variável instânciada dentro de colors.xml-->
        
        android:background="@color/primary"
        android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
```

<!--colors.xml armazena as cores selecionadas-->

### colors.xml

```python
<resources>
    <color name="purple_200">#FFBB86FC</color>
    <color name="purple_500">#FF6200EE</color>
    <color name="purple_700">#FF3700B3</color>
    <color name="teal_200">#FF03DAC5</color>
    <color name="teal_700">#FF018786</color>
    <color name="black">#FF000000</color>
    <color name="white">#FFFFFFFF</color>
    <color name="primary">#0d6efd</color>
    <color name="success">#198754</color>
</resources>
```

<!--strings.xml armazena as strings que fazem parte do display de informações-->

### strings.xml

```python
    <string name="app_name">kotlin-android-crypto-monitor</string>
    <string name="app_title">Monitor de Crypto Moedas - BITCOIN</string>
    <string name="label_rate">Cotação - BITCOIN</string>
    <string name="label_value">R$ 0,00</string>
    <string name="label_date">dd/mm/yyyy hh:mm:ss</string>
    <string name="label_refresh">Atualizar</string>
```

<!--themas.xml armazena o tema selecionado-->

### themes.xml
```python
    <style name="Theme.Kotlinandroidcryptomonitor" parent="android:Theme.Material.Light.NoActionBar" />
```
