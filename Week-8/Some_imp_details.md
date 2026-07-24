# Week 8 of #buildinpublic

This week is all about problem solving as i have covered basic oop concepts i dedicated this week for oop practice that i have learned, i did 15-20 easy and 10 medium and now doing 5 hard problems on oop. There was nothing much to share this week but at end of this week i have learned a beautiful lesson that i want to share which is, in easy wording: "Sometimes when we are stuck in a loop, everything looks perfectly logical but code breaks not giving required output, the best decision to make at this point is to close the terminal and step away" and this does work.

This happened with me so many times this week, most during solving hard problems as those require managing data flow with 3-4 dictionaries, it becomes a bit complex and when you try to solve too much in one go, at end energy goes down and the logic seeming 100% perfect to you doesn't work actually. On day 4 when i was solving a hard problem i got stuck on one, tried it, nothing made sense, closed the laptop. Next day i came, gave 2-3 hours debugging along the way and finally all tests pass.

One thing i want to share is using ai as a reviewer or code checker is very helpful, you can write your code, paste it in ai, ask it for tests for edge cases, tell it "find bugs not visible to me" and it will give you that, and knowing what's wrong going debugging it, this process i repeat 4-5 times for these problems and it was a bit annoying but was worth it at the end, that final output of terminal matching with expected results.

## The Problem

For more details here is what the problem was and where i was stuck and how i solved it.

The problem statement was:

**4. Movie Theater Booking System with Showtimes**

Context: A multiplex cinema has multiple screens (halls). Each hall has a fixed number of seats arranged in rows and columns. Customers can book seats for a specific showtime. The system must prevent double-booking and allow cancellations. Also, it should provide a real-time seat map.

Task: Create classes:

**Seat**: with attributes row, number, is_booked (private). Methods: book(), cancel().

**Hall**: with attributes name, rows, columns, seats (2D list of Seat objects). Methods:
- get_seat(row, col) returns Seat.
- get_available_seats() returns list of (row, col) tuples.

**Showtime**: with attributes movie_title, hall, start_time (string). Methods:
- book_seats(row_col_list, customer) – attempts to book multiple seats; if any unavailable, rollback? (For simplicity, book individually and return list of booked seats or None if any fail).
- cancel_booking(row_col_list, customer) – frees seats.

**Cinema**: manages multiple Hall objects and Showtime objects. Methods to add hall, add showtime, search shows by movie.

Static method: validate_seat(row, col, hall) – checks bounds.

Class variable: MAX_CANCELLATIONS_PER_CUSTOMER = 2 – maybe use it per show.

Sample Usage:
```python
hall = Hall("Screen 1", 5, 8)
show = Showtime("Avengers", hall, "18:00")
booking = show.book_seats([(1,1), (1,2)], "Alice")  # returns list of seats
show.cancel_booking([(1,1)], "Alice")  # frees one seat
```

It was basically a Cinema simulation.

## Where I Got Stuck

Here i was stuck on the book_seats method, the main flaw in the Showtime class. The problem stated in the rules: if any of the seats is not available or booked, cancel the whole request, nothing should process. But my code was stuck partially accepting what's available and still returning None if any of the seats is not found or booked, so the logic was flawed. Here is what it looked like at that time:

```python
def book_seats(self,row_col_list, customer):  # -> ([(1,1), (1,2)], "Alice")
    booked = []
    for row,col in row_col_list:
        seat = self.hall.get_seat(row,col)
        if seat is None:
            for i in booked:
                i.cancel()
            return None
    
        else:
            if not seat.status():
               seat.book()
               booked.append(seat)
    self.customers[customer] = booked   # -> Seat Objects stored in a list as values
    return booked  # -> These are seat objects
```

Now this code has many bugs, like it's updating the customer seats and overwriting previously booked ones, and partially booking seats without proper checking, meaning if you try to book two seats and the second one is already taken by someone else, the first one still gets booked and is never cancelled, even though the problem's rule clearly says the entire transaction should fail and return None.

## How I Fixed It

I sent this code to ai, it said something like this — i said "that is happening, the code logic is looking fine to me" so i shut the laptop, came back after 1 hour, said let's start from scratch with a simple calm mind. I deleted the whole method and tried a different approach, this time using the available seats method in the Hall class. I don't know why this idea didn't come before, but this was my aha moment when i said why not simply go check all of this in the available seats method first, if any fail return None, otherwise proceed. So i tried almost 5 times and after 5 attempts i had the efficient solution, this is the code that i made then:

```python
def book_seats(self,row_col_list, customer):  # -> ([(1,1), (1,2)], "Alice")
    booked = []
    all_avail = []
    available = self.hall.get_available_seats()
    for row,col in row_col_list:
        s = (row,col)
        if s in available:
            all_avail.append(s)
        else:
            return None
   
    for i,j in all_avail:
        seat = self.hall.get_seat(i,j)
        seat.book()
        booked.append(seat)
    if customer not in self.customers:
        self.customers[customer] = booked
    else:
        for i in booked:
            self.customers[customer].append(i)
    return [(i.row,i.col) for i in booked]
```

This massive jump and this clean code wasn't in one try, i got this result after 5 times, sometimes new bugs invented, sometimes very inefficient code calling functions many times, but at end finally made it work with decent efficiency.

## Full Code

The full code is on my daily code repo named codebase, also im attaching it here too:

```python
class Seat:
    def __init__(self,row,number):
        self.row = row
        self.col = number
        self.__isbooked = False

    def book(self):
        if not self.__isbooked:
            self.__isbooked = True
            return 'Seat booked'
        else:
            return 'Seat not available'

    def cancel(self):
        if self.__isbooked:
            self.__isbooked = False
            return "Seat cancelled"
        else:
            return "Seat is already free"

    def status(self):
        return self.__isbooked

class Hall:
    def __init__(self,name,row,col):
        self.name = name
        self.row = row
        self.col = col
        self.seats = []
        self.make_seats()

    def make_seats(self):
        for i in range(1,self.row+1):
            for j in range(1,self.col+1):
                t = Seat(i,j)
                self.seats.append(t)

    def get_seat(self,r,c):
        seat = (r,c)
        for i in self.seats:
            s = (i.row,i.col)
            if s == seat:
                return i
        return None

    def get_available_seats(self):
        available = []
        for i in self.seats:
            if not i.status():
                available.append((i.row,i.col))
        return available

class ShowTime:
    def __init__(self,title,hall,time):  # -> hall object 1
        self.title = title
        self.hall = hall
        self.time = time
        self.customers = {}

    def book_seats(self,row_col_list, customer):  # -> ([(1,1), (1,2)], "Alice")
        booked = []
        all_avail = []
        available = self.hall.get_available_seats()
        for row,col in row_col_list:
            s = (row,col)
            if s in available:
                all_avail.append(s)
            else:
                return None
       
        for i,j in all_avail:
            seat = self.hall.get_seat(i,j)
            seat.book()
            booked.append(seat)
        if customer not in self.customers:
            self.customers[customer] = booked
        else:
            for i in booked:
                self.customers[customer].append(i)
        return [(i.row,i.col) for i in booked]
        
        
    def cancel_booking(self,row_col_list, customer):
        if customer in self.customers:
            cancel_seats = []
            for row,col in row_col_list:
                seat = self.hall.get_seat(row,col)
                if seat is not None and seat in self.customers[customer]:
                    seat.cancel()
                    cancel_seats.append(seat)
                else:
                    return False
            
            updated_seats = []
            for i in self.customers[customer]:
                if i not in cancel_seats:
                    updated_seats.append(i)
            self.customers[customer] = updated_seats
            return True
        else:
            return False
            

class Cinema:
    def __init__(self,name):
        self.name = name
        self.halls = []
        self.shows = []

    def add_hall(self,hall):
        if hall not in self.halls:
            self.halls.append(hall)
            return "Hall added"
        else:
            return "Hall already exists in cinema"

    def add_show(self,show):
        if show not in self.shows:
            self.shows.append(show)
            return "Show added"
        else:
            return "Show already exists"  

    def search_shows_by_movie(self, movie_title):
        occurrence = []
        for i in self.shows:
            if i.title.lower().strip() == movie_title.lower().strip():
                occurrence.append((movie_title,i.time))
        return occurrence

    @staticmethod
    def validate_seat(row, col, hall):
        if hall.get_seat(row,col) is not None:
            return True
        else:
            return False

cinema = Cinema("Test")
hall = Hall("A", 3, 4)
show = ShowTime("TestMovie", hall, "12:00")
cinema.add_hall(hall); cinema.add_show(show)

print(show.book_seats([(1,1),(1,2)], "Alice"))   # Expected: [(1,1),(1,2)]
print(hall.get_available_seats())                # All except (1,1),(1,2)
print(show.cancel_booking([(1,1)], "Alice"))    # Expected: True
print(Cinema.validate_seat(2, 3, hall))          # Expected: True
print(Cinema.validate_seat(5, 1, hall))          # Expected: False
print(show.book_seats([(1,2)], "Bob"))           # Expected: [(1,1)] (now free) -> The test case sample is wrong (1,1) was never cancelled so it should be None
print(show.cancel_booking([(1,1),(1,2)], "Bob"))
```

So this was the lesson i learned, first time seeing that advice come true, and this will i think surely help in my journey along the way too. That's it for now.

Follow if you wanna see the journey.