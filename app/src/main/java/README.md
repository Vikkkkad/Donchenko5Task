# Лабораторная работа №5. Хранение данных. Настройки и внешние файлы
## Выполнила: Донченко Вика, ИСП-211о
## Язык программирования: Java

### 1. Функции приложения
Приложение предназначено для загрузки и просмотра файлов журнала "Научно-технический вестник" в формате PDF. Основные функции:
- Загрузка журнала по введенному идентификатору.
- Просмотр загруженного журнала.
- Удаление загруженного журнала.

### 2. Главный экран 
Главный экран (MainActivity) содержит следующие элементы:
- Поле для ввода идентификатора журнала.
- Кнопка "Скачать" для загрузки файла.
- Кнопка "Смотреть" для открытия загруженного файла.
- Кнопка "Удалить" для удаления загруженного файла.
- Текстовое поле для отображения статуса загрузки.

![2ск.png](..%2F..%2F..%2F..%2F2%F1%EA.png)

### 3. Загрузка файла
При нажатии на кнопку "Скачать" приложение выполняет следующие действия:
- Проверяет, введен ли идентификатор журнала.
`downloadButton.setOnClickListener(v -> {
  String journalId = journalIdInput.getText().toString();
  if (!journalId.isEmpty()) {
  createDirectoryAndDownload(journalId);
  } else {
  statusText.setText("Пожалуйста, введите номер журнала!");
  }
  });`
![3ск.png](..%2F..%2F..%2F..%2F3%F1%EA.png)
- Создает директорию для хранения загруженных файлов, если она не существует.
`private void createDirectoryAndDownload(String journalId) {
  File dir = new File(getExternalFilesDir(null), "JournalDownloads");
  if (!dir.exists()) {
  boolean isCreated = dir.mkdirs();
  if (isCreated) {
  Log.d("Directory", "Directory created successfully");
  } else {
  Log.d("Directory", "Failed to create directory");
  Toast.makeText(this, "Ошибка при создании директории!", Toast.LENGTH_SHORT).show();
  return;
  }
  }
  String fileUrl = "https://ntv.ifmo.ru/file/journal/" + journalId + ".pdf";
  new DownloadFileTask(MainActivity.this).execute(fileUrl, journalId);
  }`
- Загружает файл по заданному URL.
`private class DownloadFileTask extends AsyncTask<String, Void, Boolean> {
  private Context context;
  private String errorMessage = "";
  public DownloadFileTask(Context context) {
  this.context = context;
  }
  @Override
  protected void onPreExecute() {
  super.onPreExecute();
  statusText.setText("Файл загружается...");
  viewButton.setEnabled(false);
  deleteButton.setEnabled(false);
  }`
![4ск.png](..%2F..%2F..%2F..%2F4%F1%EA.png)
- Обновляет статус загрузки и активирует кнопки "Смотреть" и "Удалить" после успешной загрузки.
` protected void onPostExecute(Boolean success) {
  if (success) {
  viewButton.setEnabled(true);
  deleteButton.setEnabled(true);
  statusText.setText("Скачивание файла завершено!");
  } else {
  statusText.setText(errorMessage);
  }
  }`
![5ск.png](..%2F..%2F..%2F..%2F5%F1%EA.png)
- Если журнал с введенным идентификатором не найден, то выводится сообщение об ошибке.
`protected Boolean doInBackground(String... params) {
  String fileUrl = params[0];
  String journalId = params[1];
  try {
  File dir = new File(context.getExternalFilesDir(null), "JournalDownloads");
  if (!dir.exists()) {
  dir.mkdirs();
  }
  downloadedFile = new File(dir, journalId);
  URL url = new URL(fileUrl);
  HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
  urlConnection.connect();
  int responseCode = urlConnection.getResponseCode();
  if (responseCode == HttpURLConnection.HTTP_OK) {
  String contentType = urlConnection.getContentType();
  if (!"application/pdf".equals(contentType)) {
  errorMessage = "Журнал не найден!";
  return false;
  }
  InputStream inputStream = urlConnection.getInputStream();
  FileOutputStream fileOutput = new FileOutputStream(downloadedFile);
  byte[] buffer = new byte[1024];
  int bufferLength;
  while ((bufferLength = inputStream.read(buffer)) > 0) {
  fileOutput.write(buffer, 0, bufferLength);
  }
  inputStream.close();
  fileOutput.close();
  return true;
  } else {
  if (responseCode == 404) {
  errorMessage = "Журнал не найден!";
  return false;
  } else {
  errorMessage = "Ошибка: " + responseCode;
  return false;
  }
  }
  } catch (Exception e) {
  e.printStackTrace();
  errorMessage = "Ошибка при скачивании файла: " + e.getMessage();
  return false;
  }
  }`
![8ск.png](..%2F..%2F..%2F..%2F8%F1%EA.png)

### 4. Просмотр и удаление файла
- Кнопка "Смотреть" открывает загруженный PDF-файл с помощью приложения для просмотра PDF.
![6ск.png](..%2F..%2F..%2F..%2F6%F1%EA.png)
- Кнопка "Удалить" удаляет загруженный файл и обновляет интерфейс.
![7ск.png](..%2F..%2F..%2F..%2F7%F1%EA.png)

### 5.  Всплывающее окно с инструкциями
При первом запуске приложения отображается всплывающее окно с инструкциями, которые можно отключить с помощью соответствующего чекбокса.
![1ск.png](..%2F..%2F..%2F..%2F1%F1%EA.png)

