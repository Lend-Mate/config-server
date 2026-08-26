# HemenKirala Config Server

HemenKirala Config Server, mikroservislerin merkezi uygulama yapılandırmalarını sunan bir [Spring Cloud Config Server](https://spring.io/projects/spring-cloud-config) servisidir. Yapılandırma dosyalarını tek bir noktadan yönetir ve istemci servislerin profil bazlı ayarları HTTP üzerinden almasını sağlar.

## Teknolojiler

- Java 21
- Spring Boot 3.5.15
- Spring Cloud 2025.0.0
- Spring Cloud Config Server
- Spring Boot Actuator
- Maven
- Docker

## Nasıl Çalışır?

Uygulama `@EnableConfigServer` ile başlatılır ve varsayılan olarak `8888` portunu dinler.

Yapılandırma kaynağı aktif profile göre seçilir:

| Profil | Kaynak | Açıklama |
| --- | --- | --- |
| `dev` | Native filesystem | Yapılandırmalar `CONFIG_REPO_PATH` ile belirtilen yerel klasörden okunur. |
| `prod` | Git repository | Yapılandırmalar `CONFIG_GIT_URI` adresindeki Git deposundan alınır. GitHub kullanıcı adı ve token ile kimlik doğrulaması yapılır. |

Config Server endpoint formatı:

```text
/{application}/{profile}
/{application}/{profile}/{label}
```

Örnekler:

```text
http://localhost:8888/account-service/dev
http://localhost:8888/account-service/prod/main
```

Yanıt, ilgili yapılandırma dosyalarının özelliklerini Spring Cloud Config formatında döndürür.

## Gereksinimler

- JDK 21 veya üzeri
- Maven 3.9+ (veya projeyle birlikte gelen Maven Wrapper)
- `prod` profili için erişilebilir bir Git repository
- Docker ile çalıştırılacaksa Docker Engine ve Docker Compose

## Yerel Çalıştırma

`dev` profili, yapılandırma dosyalarının yerel filesystem'de bulunduğunu varsayar:

```bash
export SPRING_PROFILES_ACTIVE=dev
export CONFIG_REPO_PATH=/absolute/path/to/lendmate-config-repo
./mvnw spring-boot:run
```

Servis başladıktan sonra örnek bir isteği şu şekilde gönderebilirsiniz:

```bash
curl http://localhost:8888/account-service/dev
```

macOS/Linux dışında Maven Wrapper için `mvnw.cmd` kullanılabilir.

## Git Kaynağıyla Çalıştırma

`prod` profili Git tabanlı Config Repository kullanır. Aşağıdaki değerleri ortam değişkeni olarak sağlayın:

```bash
export SPRING_PROFILES_ACTIVE=prod
export CONFIG_GIT_URI=https://github.com/organization/config-repository.git
export GITHUB_USERNAME=your-github-username
export GITHUB_TOKEN=your-github-token
./mvnw spring-boot:run
```

Git token'ı yalnızca gerekli repository erişim izinlerine sahip olmalı ve kaynak koda yazılmamalıdır.

## Docker ile Çalıştırma

İmajı oluşturup servisi başlatmak için:

```bash
docker compose build
docker compose up -d
```

Servis `http://localhost:8888` adresinde erişilebilir olur. `docker-compose.yml`, `lendmate-net` isimli harici bir Docker network'ün mevcut olmasını bekler:

```bash
docker network create lendmate-net
docker compose up -d
```

Durdurmak için:

```bash
docker compose down
```

Docker imajı Java 21 JRE Alpine tabanlıdır, uygulamayı `spring` kullanıcısı ile çalıştırır ve 8888 portunu dışarı açar.

## Yapılandırma Değişkenleri

| Değişken | Kullanım |
| --- | --- |
| `SPRING_PROFILES_ACTIVE` | Aktif Spring profilini belirler (`dev` veya `prod`). |
| `CONFIG_REPO_PATH` | `dev` profilindeki yerel yapılandırma klasörü. |
| `CONFIG_GIT_URI` | `prod` profilindeki Git repository adresi. |
| `GITHUB_USERNAME` | Git repository erişimi için kullanıcı adı. |
| `GITHUB_TOKEN` | Git repository erişimi için token. |
| `JAVA_OPTS` | Docker ortamında JVM seçenekleri. |

Değerleri `.env` veya çalışma ortamının secret yönetimi üzerinden sağlayın. `.env` dosyasını repository'ye commit etmeyin. Gerçek erişim anahtarları veya token'lar daha önce paylaşılmışsa iptal edilip yenileri oluşturulmalıdır.

## Test

Spring context testini çalıştırmak için:

```bash
./mvnw test
```

## Proje Yapısı

```text
src/main/java/com/lendmate/configserver/
└── ConfigServerApplication.java   # Uygulama ve Config Server başlangıç noktası

src/main/resources/
└── application.yml                # Profil ve repository yapılandırması

Dockerfile                         # Çok aşamalı Docker build tanımı
docker-compose.yml                 # Container çalıştırma tanımı
pom.xml                            # Maven bağımlılıkları ve build ayarları
```

## Güvenlik Notu

Spring Boot Actuator bağımlılığı projede bulunmaktadır. Actuator endpoint'lerinin erişilebilirliği, kullanılan Spring Boot yönetim ayarlarına ve ortama bağlıdır. Üretim ortamında yönetim endpoint'lerini yalnızca gerekli olanlarla ve uygun erişim kontrolüyle açılacaktır.