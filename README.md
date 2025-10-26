# Sport_bot.
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Telegram Bot для изучения спорта
"""

import logging
import random
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import (
    Application,
    CommandHandler,
    MessageHandler,
    CallbackQueryHandler,
    ContextTypes,
    filters
)

# Настройка логирования
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

# Данные о видах спорта
SPORTS_DATA = {
    'футбол': {
        'description': 'Футбол - командный вид спорта, в котором две команды по 11 игроков пытаются забить мяч в ворота соперника.',
        'history': 'Первые упоминания о футболе датируются древним Китаем (2-3 век до н.э.). Современный футбол зародился в Англии в 19 веке.',
        'rules': 'Основные правила: игра длится 90 минут, мяч можно играть только ногами и головой, запрещён офсайд.',
        'fun_fact': 'Первый Чемпионат мира по футболу прошел в Уругвае в 1930 году.'
    },
    'баскетбол': {
        'description': 'Баскетбол - олимпийский вид спорта, в котором две команды по 5 игроков пытаются забросить мяч в кольцо соперника.',
        'history': 'Изобретён в 1891 году в США канадским преподавателем Джеймсом Нейсмитом.',
        'rules': 'Игра длится 4 периода по 10 минут, бросок сверху считается за 2 очка, трёхочковый бросок - за 3 очка.',
        'fun_fact': 'Самый высокий игрок в истории баскетбола - Сулейман Али Нашнуш (2.46 м).'
    },
    'теннис': {
        'description': 'Теннис - вид спорта, в котором игроки используют ракетки для перебрасывания мяча через сетку.',
        'history': 'Зародился во Франции в 12 веке как игра для монахов. Современный теннис появился в 19 веке в Англии.',
        'rules': 'Игра до 6 геймов, сет до 6 или 7 геймов в зависимости от турнира, матч из 3 или 5 сетов.',
        'fun_fact': 'Самая длинная партия в теннисе длилась 11 часов 5 минут на Уимблдоне в 2010 году.'
    },
    'волейбол': {
        'description': 'Волейбол - командный вид спорта, где две команды по 6 игроков перебрасывают мяч через сетку.',
        'history': 'Изобретён в 1895 году в США Уильямом Морганом как альтернатива баскетболу.',
        'rules': 'Игра до 25 очков, команда должна выиграть с преимуществом в 2 очка, матч из 3 или 5 партий.',
        'fun_fact': 'Волейболисты могут прыгать выше 3 метров в высоту.'
    },
    'хоккей': {
        'description': 'Хоккей - командный вид спорта на льду, где две команды по 6 игроков пытаются забить шайбу в ворота.',
        'history': 'Зародился в Канаде в 19 веке. Первый официальный матч состоялся в 1875 году.',
        'rules': 'Игра длится 3 периода по 20 минут, за грубые нарушения дают штрафное время.',
        'fun_fact': 'Шайба летит со скоростью до 160 км/ч при броске профессиональных игроков.'
    }
}

# Вопросы для викторины
QUIZ_QUESTIONS = [
    {
        'question': 'В каком году был изобретён баскетбол?',
        'options': ['1891', '1901', '1881', '1871'],
        'correct': 0,
        'explanation': 'Баскетбол был изобретён в 1891 году Джеймсом Нейсмитом.'
    },
    {
        'question': 'Сколько игроков в каждой команде в футболе?',
        'options': ['11', '10', '12', '9'],
        'correct': 0,
        'explanation': 'В каждой команде по 11 игроков: 10 полевых и 1 вратарь.'
    },
    {
        'question': 'Какой самый престижный теннисный турнир?',
        'options': ['Уимблдон', 'US Open', 'Roland Garros', 'Australian Open'],
        'correct': 0,
        'explanation': 'Уимблдон считается самым престижным турниром Большого шлема.'
    },
    {
        'question': 'Сколько периодов в баскетбольном матче?',
        'options': ['4', '3', '2', '5'],
        'correct': 0,
        'explanation': 'Баскетбольный матч состоит из 4 периодов по 10 минут каждый.'
    },
    {
        'question': 'В каком виде спорта используется шайба?',
        'options': ['Хоккей', 'Кёрлинг', 'Бобслей', 'Фигурное катание'],
        'correct': 0,
        'explanation': 'Шайба используется в хоккее на льду.'
    }
]

class SportLearningBot:
    def __init__(self):
        self.user_scores = {}
        self.current_quiz = {}

    async def start(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик команды /start"""
        welcome_text = """
🏆 Добро пожаловать в бот для изучения спорта! 🏆

Я помогу вам узнать много интересного о различных видах спорта:
• Футбол ⚽
• Баскетбол 🏀
• Теннис 🎾
• Волейбол 🏐
• Хоккей 🏒

Выберите действие:
/info - Информация о виде спорта
/quiz - Пройти викторину
/facts - Интересные факты
/stats - Ваша статистика
/progress - Прогресс обучения
/help - Помощь

Начнём изучать спорт вместе! 🚀
        """

        keyboard = [
            [InlineKeyboardButton("📚 Изучить спорт", callback_data='study')],
            [InlineKeyboardButton("🎯 Пройти тест", callback_data='quiz')],
            [InlineKeyboardButton("📖 Факты", callback_data='facts')],
            [InlineKeyboardButton("📊 Статистика", callback_data='stats')],
            [InlineKeyboardButton("❓ Помощь", callback_data='help')]
        ]

        reply_markup = InlineKeyboardMarkup(keyboard)

        await update.message.reply_text(welcome_text, reply_markup=reply_markup)

    async def help_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик команды /help"""
        help_text = """
🤖 Команды бота:

/start - Запуск бота и главное меню
/info <спорт> - Получить информацию о конкретном виде спорта
/quiz - Начать викторину
/facts <спорт> - Получить интересные факты о спорте
/stats - Показать вашу статистику обучения
/progress - Показать прогресс изучения
/help - Показать эту справку

Также можно использовать кнопки в меню для навигации.

Примеры:
• /info футбол
• /facts баскетбол
• /stats - посмотреть статистику
        """

        await update.message.reply_text(help_text)

    async def info_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик команды /info"""
        if not context.args:
            await update.message.reply_text(
                "Укажите вид спорта после команды /info\n"
                "Пример: /info футбол\n"
                f"Доступные виды спорта: {', '.join(SPORTS_DATA.keys())}"
            )
            return

        sport = ' '.join(context.args).lower()

        if sport not in SPORTS_DATA:
            await update.message.reply_text(
                f"Извините, информация о '{sport}' не найдена.\n"
                f"Доступные виды спорта: {', '.join(SPORTS_DATA.keys())}"
            )
            return

        info = SPORTS_DATA[sport]
        text = f"""
🏃‍♂️ {sport.upper()}

📝 Описание:
{info['description']}

📚 История:
{info['history']}

⚖️ Правила:
{info['rules']}

🎭 Интересный факт:
{info['fun_fact']}
        """

        await update.message.reply_text(text)

    async def quiz_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик команды /quiz"""
        user_id = update.effective_user.id

        # Сбрасываем текущую викторину для пользователя
        self.current_quiz[user_id] = {
            'questions': random.sample(QUIZ_QUESTIONS, min(5, len(QUIZ_QUESTIONS))),
            'current_question': 0,
            'score': 0
        }

        await self.send_next_question(update, user_id)

    async def facts_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик команды /facts"""
        if not context.args:
            await update.message.reply_text(
                "Укажите вид спорта после команды /facts\n"
                "Пример: /facts футбол\n"
                f"Доступные виды спорта: {', '.join(SPORTS_DATA.keys())}"
            )
            return

        sport = ' '.join(context.args).lower()

        if sport not in SPORTS_DATA:
            await update.message.reply_text(
                f"Извините, факты о '{sport}' не найдены.\n"
                f"Доступные виды спорта: {', '.join(SPORTS_DATA.keys())}"
            )
            return

        info = SPORTS_DATA[sport]
        await update.message.reply_text(f"🎭 Интересный факт о {sport}:\n\n{info['fun_fact']}")

    async def stats_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик команды /stats - показывает статистику пользователя"""
        user_id = update.effective_user.id

        if user_id not in self.user_scores or not self.user_scores[user_id]:
            await update.message.reply_text(
                "📊 У вас пока нет статистики.\n"
                "Пройдите несколько викторин командой /quiz чтобы увидеть свой прогресс!"
            )
            return

        scores = self.user_scores[user_id]
        total_quizzes = len(scores)
        total_questions = sum(len(self.current_quiz.get(uid, {}).get('questions', [5])) for uid in [user_id] if uid in self.current_quiz)

        if total_questions == 0:
            total_questions = total_quizzes * 5  # Предполагаем 5 вопросов в викторине

        average_score = sum(scores) / total_quizzes
        best_score = max(scores)
        total_correct = sum(scores)

        stats_text = f"""
📊 Ваша статистика обучения:

🎯 Всего викторин пройдено: {total_quizzes}
📚 Общее количество правильных ответов: {total_correct}
🏆 Лучший результат: {best_score}/5
📈 Средний результат: {average_score:.1f}/5
⭐ Процент успеха: {total_correct/total_questions*100:.1f}%

Продолжайте изучать спорт! 🚀
Используйте /quiz для новой викторины.
        """

        await update.message.reply_text(stats_text)

    async def progress_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик команды /progress - показывает прогресс изучения"""
        user_id = update.effective_user.id

        # Подсчитываем, какие виды спорта пользователь изучал
        studied_sports = set()
        facts_requested = set()

        # Эта информация будет накапливаться со временем
        # Пока что показываем базовую информацию

        progress_text = """
📈 Ваш прогресс изучения спорта:

🏆 Достижения:
• Бот запущен и готов к работе ✅
• Доступны 5 видов спорта для изучения
• Создана система викторин для проверки знаний

🎯 Рекомендации:
• Используйте /info <спорт> для изучения конкретного вида спорта
• Практикуйте знания с помощью /quiz
• Читайте интересные факты командой /facts <спорт>

🚀 Продолжайте изучать спорт регулярно для лучших результатов!
        """

        await update.message.reply_text(progress_text)

    async def button_handler(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик нажатий на кнопки"""
        query = update.callback_query
        await query.answer()

        user_id = query.from_user.id

        if query.data == 'study':
            keyboard = []
            for sport in SPORTS_DATA.keys():
                keyboard.append([InlineKeyboardButton(f"📖 {sport.title()}", callback_data=f'info_{sport}')])

            reply_markup = InlineKeyboardMarkup(keyboard)
            await query.edit_message_text(
                "Выберите вид спорта для изучения:",
                reply_markup=reply_markup
            )

        elif query.data == 'quiz':
            await self.quiz_command(query, context)

        elif query.data == 'facts':
            keyboard = []
            for sport in SPORTS_DATA.keys():
                keyboard.append([InlineKeyboardButton(f"🎭 {sport.title()}", callback_data=f'fact_{sport}')])

            reply_markup = InlineKeyboardMarkup(keyboard)
            await query.edit_message_text(
                "Выберите вид спорта для получения фактов:",
                reply_markup=reply_markup
            )

        elif query.data == 'stats':
            # Показываем статистику через команду
            user_id = query.from_user.id
            if user_id not in bot_instance.user_scores or not bot_instance.user_scores[user_id]:
                await query.edit_message_text(
                    "📊 У вас пока нет статистики.\n"
                    "Пройдите несколько викторин командой /quiz чтобы увидеть свой прогресс!"
                )
                return

            scores = bot_instance.user_scores[user_id]
            total_quizzes = len(scores)
            total_questions = total_quizzes * 5  # Предполагаем 5 вопросов в викторине

            average_score = sum(scores) / total_quizzes
            best_score = max(scores)
            total_correct = sum(scores)

            stats_text = f"""
📊 Ваша статистика обучения:

🎯 Всего викторин пройдено: {total_quizzes}
📚 Общее количество правильных ответов: {total_correct}
🏆 Лучший результат: {best_score}/5
📈 Средний результат: {average_score:.1f}/5
⭐ Процент успеха: {total_correct/total_questions*100:.1f}%

Продолжайте изучать спорт! 🚀
            """
            await query.edit_message_text(stats_text)

        elif query.data == 'help':
            help_text = """
🏆 Бот для изучения спорта помогает:

📚 Изучать различные виды спорта
🎯 Проходить интерактивные тесты
📖 Получать интересные факты
🏅 Отслеживать свой прогресс

Используйте команды:
/info <спорт> - информация о спорте
/quiz - пройти тест
/facts <спорт> - интересные факты
            """
            await query.edit_message_text(help_text)

        elif query.data.startswith('info_'):
            sport = query.data[5:]  # Убираем 'info_'
            if sport in SPORTS_DATA:
                info = SPORTS_DATA[sport]
                text = f"""
🏃‍♂️ {sport.upper()}

📝 Описание:
{info['description']}

📚 История:
{info['history']}

⚖️ Правила:
{info['rules']}

🎭 Интересный факт:
{info['fun_fact']}
                """
                await query.edit_message_text(text)

        elif query.data.startswith('fact_'):
            sport = query.data[5:]  # Убираем 'fact_'
            if sport in SPORTS_DATA:
                info = SPORTS_DATA[sport]
                await query.edit_message_text(f"🎭 Интересный факт о {sport}:\n\n{info['fun_fact']}")

    async def send_next_question(self, update_or_query, user_id):
        """Отправляет следующий вопрос викторины"""
        quiz = self.current_quiz.get(user_id)

        if not quiz or quiz['current_question'] >= len(quiz['questions']):
            # Викторина завершена
            score = quiz['score']
            total = len(quiz['questions'])

            if update_or_query.callback_query:
                await update_or_query.callback_query.edit_message_text(
                    f"🎉 Викторина завершена!\n\n"
                    f"Ваш результат: {score}/{total}\n"
                    f"Процент правильных ответов: {score/total*100:.1f}%"
                )
            else:
                await update_or_query.message.reply_text(
                    f"🎉 Викторина завершена!\n\n"
                    f"Ваш результат: {score}/{total}\n"
                    f"Процент правильных ответов: {score/total*100:.1f}%"
                )

            # Сохраняем статистику
            if user_id not in self.user_scores:
                self.user_scores[user_id] = []
            self.user_scores[user_id].append(score)

            return

        question_data = quiz['questions'][quiz['current_question']]

        keyboard = []
        for i, option in enumerate(question_data['options']):
            keyboard.append([InlineKeyboardButton(option, callback_data=f'answer_{user_id}_{i}')])

        reply_markup = InlineKeyboardMarkup(keyboard)

        question_text = f"Вопрос {quiz['current_question'] + 1}/{len(quiz['questions'])}\n\n{question_data['question']}"

        if update_or_query.callback_query:
            await update_or_query.callback_query.edit_message_text(
                text=question_text,
                reply_markup=reply_markup
            )
        else:
            await update_or_query.message.reply_text(
                text=question_text,
                reply_markup=reply_markup
            )

    async def answer_handler(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик ответов на вопросы викторины"""
        query = update.callback_query
        await query.answer()

        # Парсим callback_data: answer_{user_id}_{answer_index}
        parts = query.data.split('_')
        if len(parts) != 3:
            return

        user_id = int(parts[1])
        answer_index = int(parts[2])

        quiz = self.current_quiz.get(user_id)
        if not quiz:
            return

        question_data = quiz['questions'][quiz['current_question']]

        if answer_index == question_data['correct']:
            quiz['score'] += 1
            feedback = "✅ Правильно!"
        else:
            feedback = f"❌ Неправильно. Правильный ответ: {question_data['options'][question_data['correct']]}"

        feedback += f"\n\n{question_data['explanation']}"

        quiz['current_question'] += 1

        # Отправляем обратную связь и следующий вопрос
        await query.edit_message_text(feedback)

        # Ждем секунду перед следующим вопросом
        import asyncio
        await asyncio.sleep(1)

        await self.send_next_question(query, user_id)

    async def text_handler(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик текстовых сообщений"""
        text = update.message.text.lower()

        if text in ['привет', 'hello', 'hi']:
            await update.message.reply_text(
                "Привет! 👋 Я бот для изучения спорта.\n"
                "Используйте /start для начала работы или /help для помощи."
            )
        elif text in ['спасибо', 'thanks', 'thank you']:
            await update.message.reply_text("Пожалуйста! 😊 Продолжайте изучать спорт!")
        else:
            await update.message.reply_text(
                "Я не понимаю это сообщение. Используйте /help для получения списка команд."
            )

def main():
    """Основная функция"""
    # Здесь нужно указать токен вашего бота
    # Получить токен можно у @BotFather в Telegram
    TOKEN = "8175305515:AAHlK4SJeZ4Z2bNkY9nJd-s79ifwmPpcznw"

    # Проверяем валидность токена (должен содержать двоеточие и быть длиной около 45 символов)
    if ':' not in TOKEN or len(TOKEN) < 35:
        print("❌ ОШИБКА: Токен бота указан некорректно!")
        print(f"Текущий токен: {TOKEN}")
        print("\n📝 Инструкция:")
        print("1. Перейдите к @BotFather в Telegram")
        print("2. Отправьте команду /newbot")
        print("3. Придумайте имя и username для бота")
        print("4. Скопируйте полученный токен")
        print("5. Убедитесь, что токен в кавычках и содержит двоеточие")
        print("\n🧪 Для тестирования функционала без Telegram используйте:")
        print("python -c \"from sport_bot import *; print('База знаний загружена успешно!')\"")
        return

    # Создаем приложение
    application = Application.builder().token(TOKEN).build()

    # Создаем экземпляр бота
    bot_instance = SportLearningBot()

    # Регистрируем обработчики
    application.add_handler(CommandHandler("start", bot_instance.start))
    application.add_handler(CommandHandler("help", bot_instance.help_command))
    application.add_handler(CommandHandler("info", bot_instance.info_command))
    application.add_handler(CommandHandler("quiz", bot_instance.quiz_command))
    application.add_handler(CommandHandler("facts", bot_instance.facts_command))
    application.add_handler(CommandHandler("stats", bot_instance.stats_command))
    application.add_handler(CommandHandler("progress", bot_instance.progress_command))

    # Обработчик нажатий на кнопки
    application.add_handler(CallbackQueryHandler(bot_instance.button_handler))
    application.add_handler(CallbackQueryHandler(bot_instance.answer_handler, pattern=r'^answer_'))

    # Обработчик текстовых сообщений
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, bot_instance.text_handler))

    print("🚀 Бот запущен! Отправьте /start в Telegram для начала работы.")
    print("📊 Доступные виды спорта:", list(SPORTS_DATA.keys()))
    print("❓ Количество вопросов в викторине:", len(QUIZ_QUESTIONS))

    # Запускаем бота
    application.run_polling()

if __name__ == '__main__':
    main()
