**入口**
SpringBoot1.5, 通过实例化SpringApplication启动
```java
@SpringBootApplication
public class Application {

	/**
	 * Logger
	 */
	private static final Logger LOGGER = LoggerFactory.getLogger(Application.class);

	/**
	 * Entry of app.
	 *
	 * @param args
	 */
	public static void main(String[] args) {
		try {
			SpringApplication application = new SpringApplication(Application.class);
			application.setBannerMode(Banner.Mode.OFF);
			ConfigurableApplicationContext applicationContext = application.run(args);
		} catch (Exception e) {
			LOGGER.error("Unexpected exception happen when starting up SpringBoot application! ", e);
			System.exit(-1);

		}
	}
}
```

**初始化BeanFactoryPostProcessor**

