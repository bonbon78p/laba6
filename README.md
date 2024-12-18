# Лабораторная работа №6. Уведомления.
Студент: Перцов Богдан, ИСП-221С

Приложение позволяет пользователю вводить заголовок и текст напоминания, выбирать дату и время срабатывания, а также просматривать список созданных напоминаний. Напоминания сохраняются в локальной базе данных. При срабатывании напоминания, пользователю показывается уведомление

## 1. Функциональность

• Добавление напоминаний: Пользователь вводит заголовок и текст напоминания, выбирает дату и время с помощью встроенных диалогов выбора. Данные сохраняются в базе данных, и одновременно устанавливается будильник с помощью AlarmManager для отображения уведомления в указанное время.

• Просмотр напоминаний: Список всех созданных напоминаний отображается в RecyclerView. Каждый элемент списка содержит заголовок, текст, дату и время, а также кнопку "Удалить".

• Удаление напоминаний: Нажатие кнопки "Удалить" удаляет запись из базы данных и отменяет соответствующий будильник, установленный ранее с помощью AlarmManager.

• Уведомления: При срабатывании будильника, ReminderReceiver отображает уведомление с использованием NotificationManager. Уведомление содержит заголовок и текст напоминания. Нажатие на уведомление запускает основное приложение.

• Обработка разрешений: Приложение запрашивает разрешение POST_NOTIFICATIONS и отображает сообщение пользователю, если разрешение не предоставлено.
## 2. Архитектура

Приложение состоит из следующих основных компонентов:

• MainActivity: Основная активити, которая управляет пользовательским интерфейсом. Она содержит поля ввода текста, кнопки для выбора даты и времени, кнопку сохранения, и RecyclerView для отображения списка напоминаний. MainActivity взаимодействует с DatabaseHelper для работы с базой данных и с AlarmManager для управления будильниками.
```
buttonSave.setOnClickListener(v -> {
    String title = editTextTitle.getText().toString();
    String message = editTextMessage.getText().toString();
    Calendar calendar = Calendar.getInstance();
    calendar.set(selectedYear, selectedMonth, selectedDay, selectedHour, selectedMinute);
    long triggerTime = calendar.getTimeInMillis();
    if (dbHelper.addReminder(title, message, triggerTime)) {
        reminderList.add(new Reminder(title, message, new SimpleDateFormat("yyyy-MM-dd HH:mm", Locale.getDefault()).format(calendar.getTime())));
        reminderAdapter.notifyDataSetChanged();
        setAlarm(title, message, triggerTime);
    } else {
        Toast.makeText(this, "Ошибка сохранения", Toast.LENGTH_SHORT).show();
    }
});
```

• DatabaseHelper: Класс, реализующий SQLiteOpenHelper, для взаимодействия с базой данных SQLite. Он отвечает за создание таблицы, добавление, получение и удаление записей.
```
public boolean addReminder(String title, String message, long triggerTime) {
    SQLiteDatabase db = this.getWritableDatabase();
    ContentValues values = new ContentValues();
    values.put(COL_2, title);
    values.put(COL_3, message);
    values.put(COL_4, new SimpleDateFormat("yyyy-MM-dd HH:mm", Locale.getDefault()).format(new Date(triggerTime)));
    long result = db.insert(TABLE_NAME, null, values);
    db.close();
    return result != -1;
}
```

• Reminder: Класс-модель данных, представляющий одно напоминание. Содержит поля: id (целое число, первичный ключ), title (строка), message (строка), date (строка, хранит дату и время в формате "yyyy-MM-dd HH:mm").
```
public class Reminder {
    public int id;
    public String title;
    public String message;
    public String date;

}
```

• ReminderAdapter: Адаптер для RecyclerView, который отображает список напоминаний. Он включает обработчик клика по кнопке "Удалить", который вызывает метод onDeleteClick в MainActivity.
```
holder.deleteButton.setOnClickListener(v -> {
    onDeleteClickListener.onDeleteClick(reminder);
});
```

• ReminderReceiver: BroadcastReceiver, который получает сообщение от AlarmManager и отображает уведомление с помощью NotificationManager. При нажатии на уведомление, запускается MainActivity.
```
NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
    .setSmallIcon(R.drawable.ic_launcher_foreground) // Замените на вашу иконку
    .setContentTitle(title)
    .setContentText(message)
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .setAutoCancel(true)
    .setContentIntent(pendingIntent); // PendingIntent для запуска MainActivity

NotificationManagerCompat notificationManager = NotificationManagerCompat.from(context);
notificationManager.notify(NOTIFICATION_ID, builder.build());
```

## Сборка и запуск

1. Клонирование: Клонируйте репозиторий с помощью git clone [URL репозитория].
2. Открытие проекта: Откройте каждый проект (l31 и l32) в Android Studio.
3. Запуск: Запустите каждое приложение на эмуляторе или устройстве Android.
