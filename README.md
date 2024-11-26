import sqlite3


class DatabaseManager:
    def init(self, db_name):
        """Инициализация с указанием имени базы данных"""
        self.db_name = db_name
        self.connection = None

    def open_connection(self):
        """Открытие соединения с базой данных"""
        self.connection = sqlite3.connect(self.db_name)

    def close_connection(self):
        """Закрытие соединения с базой данных"""
        if self.connection:
            self.connection.close()

    def find_user_by_name(self, username):
        """Поиск пользователя по имени"""
        cursor = self.connection.cursor()
        cursor.execute("SELECT * FROM users WHERE username = ?", (username,))
        return cursor.fetchone()

    def execute_transaction(self, operations):
        """Выполнение нескольких операций в одной транзакции"""
        try:
            cursor = self.connection.cursor()
            for operation in operations:
                cursor.execute(*operation)
            self.connection.commit()
        except sqlite3.Error as e:
            self.connection.rollback()
            raise RuntimeError(f"Ошибка при выполнении транзакции: {e}")


class User:
    def init(self, db_manager):
        """Инициализация с указанием менеджера базы данных"""
        self.db_manager = db_manager

    def create_table(self):
        """Создание таблицы users"""
        cursor = self.db_manager.connection.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT UNIQUE,
                email TEXT
            )
        """)
        self.db_manager.connection.commit()

    def add_user(self, username, email):
        """Добавление нового пользователя"""
        cursor = self.db_manager.connection.cursor()
        cursor.execute("INSERT INTO users (username, email) VALUES (?, ?)", (username, email))
        self.db_manager.connection.commit()

    def get_user_by_id(self, user_id):
        """Получение пользователя по ID"""
        cursor = self.db_manager.connection.cursor()
        cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
        return cursor.fetchone()

    def delete_user(self, user_id):
        """Удаление пользователя по ID"""
        cursor = self.db_manager.connection.cursor()
        cursor.execute("DELETE FROM users WHERE id = ?", (user_id,))
        self.db_manager.connection.commit()


class Admin(User):
    def create_table(self):
        """Создание таблицы admins"""
        cursor = self.db_manager.connection.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS admins (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT UNIQUE,
                role TEXT
            )
        """)
        self.db_manager.connection.commit()

    def add_admin(self, username, role):
        """Добавление нового администратора"""
        cursor = self.db_manager.connection.cursor()
        cursor.execute("INSERT INTO admins (username, role) VALUES (?, ?)", (username, role))
        self.db_manager.connection.commit()


class Customer(User):
    def create_table(self):
        """Создание таблицы customers"""
        cursor = self.db_manager.connection.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS customers (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT UNIQUE,
                address TEXT
            )
        """)
        self.db_manager.connection.commit()

    def add_customer(self, username, address):
        """Добавление нового клиента"""
        cursor = self.db_manager.connection.cursor()
        cursor.execute("INSERT INTO customers (username, address) VALUES (?, ?)", (username, address))
        self.db_manager.connection.commit()


if __name__ == "__main__":
    db_manager = DatabaseManager("example.db")
    db_manager.open_connection()

    user_manager = User(db_manager)
    user_manager.create_table()
    user_manager.add_user("john_doe", "john@example.com")
    print(user_manager.get_user_by_id(1))

    admin_manager = Admin(db_manager)
    admin_manager.create_table()
    admin_manager.add_admin("admin_user", "superuser")

    customer_manager = Customer(db_manager)
    customer_manager.create_table()
    customer_manager.add_customer("customer1", "123 Main St")

    db_manager.close_connection()
