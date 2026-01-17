# Простая система бронирования
# Сделано как будто студентом

class User:
    def __init__(self, user_id, name):
        self.id = user_id
        self.name = name

class Seat:
    FREE = "свободно"
    RESERVED = "забронировано"
    SOLD = "продано"
    
    def __init__(self, seat_id):
        self.id = seat_id
        self.status = self.FREE
        self.user = None
    
    def __str__(self):
        user_name = self.user.name if self.user else "нет"
        return f"Место {self.id}: {self.status}, занято: {user_name}"

class EventSession:
    def __init__(self, session_id, time):
        self.id = session_id
        self.time = time
        self.seats = {}
        
        # создаем 6 мест
        for i in range(1, 7):
            seat_id = f"A{i}"
            self.seats[seat_id] = Seat(seat_id)
    
    def show_seats(self):
        print(f"\nСеанс: {self.id}, время: {self.time}")
        print("Места в зале:")
        for seat in self.seats.values():
            print(f"  {seat}")
    
    def get_seat(self, seat_id):
        return self.seats.get(seat_id)

# команды (простые функции, не полноценный паттерн)
class BookingSystem:
    def __init__(self):
        self.history = []
    
    def reserve_seat(self, session, seat_id, user):
        seat = session.get_seat(seat_id)
        if seat and seat.status == Seat.FREE:
            # запоминаем старое состояние
            old_state = (seat.status, seat.user)
            self.history.append(("reserve", seat_id, old_state))
            
            # меняем
            seat.status = Seat.RESERVED
            seat.user = user
            print(f"{user.name} забронировал место {seat_id}")
            return True
        else:
            print(f"Не удалось забронировать {seat_id}")
            return False
    
    def cancel_reservation(self, session, seat_id, user):
        seat = session.get_seat(seat_id)
        if seat and seat.status == Seat.RESERVED and seat.user and seat.user.id == user.id:
            old_state = (seat.status, seat.user)
            self.history.append(("cancel", seat_id, old_state))
            
            seat.status = Seat.FREE
            seat.user = None
            print(f"{user.name} отменил бронь места {seat_id}")
            return True
        else:
            print(f"Не удалось отменить бронь {seat_id}")
            return False
    
    def buy_ticket(self, session, seat_id, user):
        seat = session.get_seat(seat_id)
        if seat and seat.status == Seat.RESERVED and seat.user and seat.user.id == user.id:
            old_state = (seat.status, seat.user)
            self.history.append(("buy", seat_id, old_state))
            
            seat.status = Seat.SOLD
            print(f"{user.name} купил билет на место {seat_id}")
            return True
        else:
            print(f"Не удалось купить билет на {seat_id}")
            return False
    
    def undo_last(self, session):
        if not self.history:
            print("Нечего отменять")
            return
        
        last_action = self.history.pop()
        action_type, seat_id, old_state = last_action
        seat = session.get_seat(seat_id)
        
        if seat:
            old_status, old_user = old_state
            seat.status = old_status
            seat.user = old_user
            print(f"Отмена: {action_type} для места {seat_id}")

# главная программа
def main():
    print("система бронирования билетов")
    print("=" * 30)
    
    # создаем пользователей
    user1 = User("1", "иван")
    user2 = User("2", "мария")
    
    # создаем сеанс
    movie = EventSession("фильм 'матрица'", "20:00")
    
    # создаем систему бронирования
    system = BookingSystem()
    
    # показываем начальное состояние
    print("\nначальное состояние:")
    movie.show_seats()
    
    # тестируем
    print("\n--- тест 1: бронирование ---")
    system.reserve_seat(movie, "A1", user1)
    system.reserve_seat(movie, "A2", user2)
    movie.show_seats()
    
    print("\n--- тест 2: покупка билета ---")
    system.buy_ticket(movie, "A1", user1)
    movie.show_seats()
    
    print("\n--- тест 3: отмена брони ---")
    system.cancel_reservation(movie, "A2", user2)
    movie.show_seats()
    
    print("\n--- тест 4: отмена последней операции ---")
    system.undo_last(movie)
    movie.show_seats()
    
    print("\n--- тест 5: ошибки ---")
    system.buy_ticket(movie, "A3", user1)  # место свободное
    system.reserve_seat(movie, "A1", user2)  # место уже занято
    
    print("\nконечное состояние:")
    movie.show_seats()
    
    print("\nвсе операции завершены")

if __name__ == "__main__":
    main()
