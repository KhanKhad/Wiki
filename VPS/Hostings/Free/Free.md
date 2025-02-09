Тут регулярно кто-нибудь хочет раздать несколько серверов 3x-ui в одной подписке. Можно так:
- называем подписку клиента одинаково на всех серверах 
- берем скрипт:


<?php

function fetch_url_content($url) { // Функция считывает подписку по заданному адресу
  $context = stream_context_create([ // Создаем контекст с опцией игнорирования ошибок сертификата
        'ssl' => [
            'verify_peer' => false,
            'verify_peer_name' => false,
        ],
    ]);
    $content = @file_get_contents($url, false, $context); // Читаем подписку (@ для подавления ошибок)
    if ($content === FALSE) {
        return []; // Возвращаем пустой массив в случае ошибки
    }
  //return explode("\n", $content); // Разделяем содержимое на строки (используйте, если подписки не закодированы)
  return explode("\n", base64_decode($content)); // Декодируем и разделяем содержимое на строки (если подписки не закодированы, используйте предыдущую строку)
}

function combine_unique_lines($urls) {  // Функция объединяет содержимое всех подписок
  $lines = [];
  foreach ($urls as $url) {
        $lines = array_merge($lines, fetch_url_content($url));
    }
  
  $lines = array_filter($lines); // Удаляем пустые строки
    return array_unique($lines); // Удаляем дубликаты
}


$urls = [ // !!!! Задаем список URL, подписки с которых мы хотим объединить !!!!
  'https://123.123.123.123:12345/path1/',
  'https://[2222:2222:2222::2222]:6789/path2/',
  'https://example.xyz/path3/',
  // и т. д.
];  

$queryString = $_SERVER['QUERY_STRING']; // Считываем параметры запроса (то, что в адресе после знака ?), там должно быть название конкретной подписки

foreach ($urls as &$url) {
    $url .= $queryString; // Добавляем название подписки к каждому URL
 }

$unique_lines = combine_unique_lines($urls); // Считываем подписки
$output = base64_encode(implode("\n", $unique_lines)); // Объединяем строки с разделителем, кодируем base64

header('Content-Type: text/plain'); // Устанавливаем Content-Type
header('profile-update-interval: 7'); // Задаем частоту обновления подписки
//header('subscription-userinfo: upload=0; download=0; total=0; expire=0'); // Если раскомментить, hiddify будет писать, что число дней подписки бесконечно
header('profile-title: ' . $queryString); // Задаем название подписки

echo $output;

?>


- редактируем: вставляем в него адреса своих серверов
- закидываем его на любой бесплатный хостинг с php 
- даем клиенту ссылку вида https://hosting/path/script.php?subs, где subs – его подписка
UPD. На бесплатных тарифах, оказывается, часто блокируют доступ для приложений, так что надо проверять хостера. Точно работает с alwaysdata. Ну и на своем vps никто не запрещает поднять сервер с php.

Alwaysdata: регаемся, включаем ssh доступ, подключаемся, в директорию /home/<username>/www кидаем этот скрипт и потом можем получить его по ссылке https://<username>.alwaysdata.net/script.php?<user>