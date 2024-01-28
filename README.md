# final.sql



```sql
--
-- Anamakine: 127.0.0.1:3306
-- Üretim Zamanı: 26 Oca 2024, 22:06:05
-- Sunucu sürümü: 8.0.31
-- PHP Sürümü: 8.0.26

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";


/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

--
-- Veritabanı: `final`
--

-- --------------------------------------------------------

--
-- Tablo için tablo yapısı `forum`
--

DROP TABLE IF EXISTS `forum`;
CREATE TABLE IF NOT EXISTS `forum` (
  `kod` int NOT NULL AUTO_INCREMENT,
  `kim_yükledi` int NOT NULL,
  `yorum` text COLLATE utf8mb4_general_ci NOT NULL,
  `tarih` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`kod`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

-- --------------------------------------------------------

--
-- Tablo için tablo yapısı `kullanici`
--

DROP TABLE IF EXISTS `kullanici`;
CREATE TABLE IF NOT EXISTS `kullanici` (
  `kod` int NOT NULL AUTO_INCREMENT,
  `kullaniciadi` varchar(255) COLLATE utf8mb4_general_ci NOT NULL,
  `sifre` varchar(255) COLLATE utf8mb4_general_ci NOT NULL,
  PRIMARY KEY (`kod`)
) ENGINE=MyISAM AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

--
-- Tablo döküm verisi `kullanici`
--

INSERT INTO `kullanici` (`kod`, `kullaniciadi`, `sifre`) VALUES
(1, 'eyup', '$2y$10$fp/rZSqvKziQQIxot.6MT.N0cPWf8LetiiaNXP3fjsV8B36BGR83y');
COMMIT;

/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;

```


# index.php
```html
<!DOCTYPE html>
<html lang="tr">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Forum</title>
    <link rel="stylesheet" href="http://localhost/classroom/stil/stil.css">
</head>

<body>
    
    <ul class="ustmenu">
    
    <li><a href="oku.php">Oku</a></li>
    <li><a href="yaz.php">Yaz</a></li>
    <li><a href="ara.php">Ara</a></li>
    <li><a href="cikis.php">Sil</a></li>
    </ul>
    
    

</body>

</html>

```


# oku.php
```php
<!DOCTYPE html>
<html lang="tr">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Oku</title>
    <link rel="stylesheet" href="http://localhost/classroom/stil/stil.css">
    <script>
        function geriDon() {
            window.location.href = 'index.php';
        }

        function uyariVeYonlendir() {
            alert("Giriş Yapmalısınız!");
            window.location.href = 'giris.php';
        }
    </script>
</head>

<body>
    <?php
    session_start();
    if (isset($_SESSION['kullanici'])) {
        echo "<p>Hoşgeldiniz, " . htmlspecialchars($_SESSION['kullanici']) . "!</p>";
        echo "<button onclick='geriDon()'>Geri Dön</button>";
    } else {
        echo "<script>uyariVeYonlendir();</script>";
    }
    ?>
</body>

</html>
```

# yaz.php
```php
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Yorum Yaz</title>
    <script>
        function girisYapUyarisi() {
            alert("Yorum yazmak için öncelikle giriş yapmalısınız!");
            window.location.href = 'giris.php';
        }
    </script>
</head>
<body>
    <?php
    session_start();
    if (isset($_SESSION['kullanici'])) {
        // Yorum formunu göster
        echo '<form action="yorumKaydet.php" method="post">';
        echo '<textarea name="yorum" rows="4" cols="50"></textarea><br>';
        echo '<input type="submit" value="Kaydet">';
        echo '</form>';
        echo '<a href="index.php">Geri Dön</a>';

    } else {
        // Kullanıcı giriş yapmamışsa uyarı ver ve giris.php sayfasına yönlendir
        echo '<script>girisYapUyarisi();</script>';
    }
    ?>
</body>
</html>

```

# yorumKaydet.php
```php
<?php
session_start();

// Oturum kontrolü
if (!isset($_SESSION['kullanici'])) {
    echo "Giriş yapmanız gerekmektedir.";
    header("Refresh:2; url=giris.php");
    exit;
}

// Veritabanı bağlantı bilgileri
$host = 'localhost'; 
$dbname = 'final'; 
$user = 'root'; 
$password = ''; 

try {
    $vt = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8", $user, $password);
    $vt->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    // Formdan gelen yorumu al
    $yorum = trim($_POST['yorum']);
    $kimYukledi = $_SESSION['kullanici'];  // Kullanıcı adını oturumdan al

    // Yorumun boş olup olmadığını kontrol et
    if (empty($yorum)) {
        echo "Yorum alanı boş bırakılamaz!";
        header("Refresh:2; url=yaz.php");
        exit;
    }

    // SQL sorgusunu hazırla
    $sorgu = $vt->prepare("INSERT INTO forum (kim_yükledi, yorum, tarih) VALUES (:kim_yukledi, :yorum, NOW())");
    $sorgu->bindParam(':kim_yukledi', $kimYukledi);
    $sorgu->bindParam(':yorum', $yorum);

    // Sorguyu çalıştır
    $sorgu->execute();

    echo "Yorumunuz başarıyla kaydedildi.";
    header("Refresh:2; url=index.php");

} catch (PDOException $e) {
    echo "Veritabanı hatası: " . $e->getMessage();
    die();
}

$vt = null;
?>

```

# ara.php
```php
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Arama Sayfası</title>
    <script>
        function kullaniciBulunamadi() {
            alert("Aradığınız kullanıcı bulunamamıştır!");
        }
        function girisYapUyarisi() {
            alert("Arama yapmak için öncelikle giriş yapmalısınız!");
            window.location.href = 'giris.php';
        }

    </script>
</head>
<body>
    <?php
    session_start();
    if (isset($_SESSION['kullanici'])) {
        // Arama yapma formunu göster
        echo '<form action="ara.php" method="post">';
        echo '<input type="text" name="kullaniciadi" placeholder="Kullanıcı adı girin">';
        echo '<input type="submit" value="Arama Yap"><br>';
        echo '</form>';
        echo '<a href="index.php">Geri Dön</a>';

    } else {
        // Kullanıcı giriş yapmamışsa uyarı ver ve giris.php sayfasına yönlendir
        echo '<script>girisYapUyarisi();</script>';
    }
    ?>

    <?php
    if ($_SERVER["REQUEST_METHOD"] == "POST") {
        $host = 'localhost';
        $dbname = 'final';
        $user = 'root';
        $password = '';

        try {
            $vt = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8", $user, $password);
            $vt->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

            $arananKullanici = $_POST['kullaniciadi'];

            $sorgu = $vt->prepare("SELECT * FROM forum WHERE kim_yükledi = :kullaniciadi");
            $sorgu->bindParam(':kullaniciadi', $arananKullanici);
            $sorgu->execute();

            if ($sorgu->rowCount() > 0) {
                while ($row = $sorgu->fetch(PDO::FETCH_ASSOC)) {
                    echo "<p>" . htmlspecialchars($row['yorum']) . "</p>";
                }
            } else {
                echo "<script>kullaniciBulunamadi();</script>";
            }

        } catch (PDOException $e) {
            echo "Veritabanı hatası: " . $e->getMessage();
            die();
        }

        $vt = null;
    }
    ?>
</body>
</html>

```

# cikis.php
```php
<?php
session_start(); // Oturumu başlat

// Tüm oturum değişkenlerini temizle
$_SESSION = array();

// Oturumu sonlandır
session_destroy();

// Kullanıcıyı ana sayfaya yönlendir
echo "Çıkış başarılı!";
header("Refresh:1; url=index.php");
exit;
?>

```

# giris.php
```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Giriş Yap</title>
</head>
<body>
    <form method="post" action="girisKontrol.php">
        <label for="kullanici">Kullanıcı Adı:</label>
        <input type="text" name="kullanici" id="kullanici" required>
        <br><br>
        <label for="sifre">Şifre:</label>
        <input type="password" name="sifre" id="sifre" required>
        <br><br>
        <input type="submit" value="Giriş Yap">
    </form>
        <br><br>
        <br><br>
    <form method="post" action="kayitSayfasi.php">
        <input type="submit" value="Kayıt Sayfasına Git =>">
    </form>

</body>
</html>

```

# girisKontrol.php
```php
<?php
// Veritabanı bağlantı bilgileri
$host = 'localhost'; 
$dbname = 'final'; 
$user = 'root'; 
$password = ''; 

try {
    // Veritabanına bağlan
    $vt = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8", $user, $password);
    $vt->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    // Formdan gelen verileri al
    $kullaniciadi = $_POST['kullanici'];
    $sifre = $_POST['sifre'];

    // Kullanıcı bilgilerini kontrol et
    $sorgu = $vt->prepare("SELECT sifre FROM kullanici WHERE kullaniciadi = :kullaniciadi");
    $sorgu->bindParam(':kullaniciadi', $kullaniciadi);
    $sorgu->execute();

    if ($sorgu->rowCount() > 0) {
        $row = $sorgu->fetch(PDO::FETCH_ASSOC);
        if (password_verify($sifre, $row['sifre'])) {
            session_start();
            $_SESSION['kullanici'] = $kullaniciadi;  // Kullanıcı adını oturuma kaydet
            echo "Tebrikler, giriş yaptınız!";
            header("Refresh:2; url=index.php");
        } else {
            echo "<script>alert('Girmiş olduğunuz bilgiler hatalıdır!'); window.history.back();</script>";
        }
    } else {
        echo "<script>alert('Girmiş olduğunuz bilgiler hatalıdır!'); window.history.back();</script>";
    }
} catch (PDOException $e) {
    echo "Veritabanı hatası: " . $e->getMessage();
    die();
}

// Veritabanı bağlantısını kapat
$vt = null;
?>

```

# kayitSayfasi.php
```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kaydol</title>
</head>
<body>
    <form method="post" action="add_user.php">
        <label for="kullanici">Kullanıcı Adı:</label>
        <input type="text" name="kullanici" id="kullanici" required>
        <br><br>
        <label for="sifre">Şifre:</label>
        <input type="password" name="sifre" id="sifre" required>
        <br><br>
        <input type="submit" value="Kaydol">
    </form>
</body>
</html>

```

# add_user.php
```php
<?php
// Veritabanı bağlantı bilgileri
$host = 'localhost'; 
$dbname = 'final'; 
$user = 'root'; 
$password = ''; 

try {
    // Veritabanına bağlan
    $vt = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8", $user, $password);
    $vt->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    if (isset($_POST['kullanici']) && isset($_POST['sifre'])) {
        $kullaniciadi = $_POST['kullanici'];
        $sifre = $_POST['sifre'];

        // Kullanıcı adı uzunluğunu kontrol et
        if (strlen($kullaniciadi) < 3) {
            echo "<script>alert('Girmiş olduğunuz kullanıcı adı en az 3 karakterli olmalıdır!'); window.history.back();</script>";
            exit;
        }

        // Kullanıcı adının zaten kullanılıp kullanılmadığını kontrol et
        $sorgu = $vt->prepare("SELECT * FROM kullanici WHERE kullaniciadi = :kullaniciadi");
        $sorgu->bindParam(':kullaniciadi', $kullaniciadi);
        $sorgu->execute();

        if ($sorgu->rowCount() > 0) {
            // Kullanıcı adı zaten mevcut
            echo "<script>alert('Bu kullanıcı adı daha önce kullanıldı!'); window.history.back();</script>";
            exit;
        }

        // Şifreyi hashle
        $hashlenmisSifre = password_hash($sifre, PASSWORD_DEFAULT);

        // Kullanıcıyı veritabanına ekle
        $sorgu = $vt->prepare("INSERT INTO kullanici (kullaniciadi, sifre) VALUES (:kullaniciadi, :sifre)");
        $sorgu->bindParam(':kullaniciadi', $kullaniciadi);
        $sorgu->bindParam(':sifre', $hashlenmisSifre);
        $sorgu->execute();

        echo "Kullanıcı başarıyla kaydedildi.";
        header('Refresh:2; url=giris.php');
    } else {
        // Kullanıcı adı veya şifre yoksa bir hata mesajı göster
        echo "Kullanıcı adı ve şifre alanları doldurulmalıdır.";
    }
} catch (PDOException $e) {
    echo "Veritabanı hatası: " . $e->getMessage();
    die();
}

// Veritabanı bağlantısını kapat
$vt = null;
?>

```
