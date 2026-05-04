# Hotel-Reservation-System-C-
#include <iostream>
#include <fstream>
#include <cstring>
using namespace std;

class Hotel {
private:
    int room_no;
    char name[50];
    int days;
    float bill;

public:
    void bookRoom() {
        cout << "\nEnter Room Number: ";
        cin >> room_no;
        cin.ignore();

        cout << "Enter Customer Name: ";
        cin.getline(name, 50);

        cout << "Enter Days of Stay: ";
        cin >> days;

        bill = days * 1000; // fixed rate per day
    }

    void display() {
        cout << "\nRoom No: " << room_no;
        cout << "\nCustomer Name: " << name;
        cout << "\nDays: " << days;
        cout << "\nTotal Bill: Rs. " << bill << endl;
    }

    int getRoomNo() {
        return room_no;
    }
};

// 🔐 LOGIN SYSTEM
bool login() {
    string user, pass;
    cout << "\n--- Admin Login ---\n";
    cout << "Username: ";
    cin >> user;
    cout << "Password: ";
    cin >> pass;

    if (user == "admin" && pass == "1234") {
        cout << "\nLogin Successful!\n";
        return true;
    } else {
        cout << "\nInvalid Credentials!\n";
        return false;
    }
}

// 🏨 CHECK ROOM AVAILABILITY
bool isRoomAvailable(int room) {
    Hotel h;
    ifstream file("records.dat");

    while (file.read((char*)&h, sizeof(h))) {
        if (h.getRoomNo() == room)
            return false;
    }

    return true;
}

// ➕ BOOK ROOM
void addBooking() {
    Hotel h;
    int room;

    cout << "\nEnter Room Number: ";
    cin >> room;

    if (!isRoomAvailable(room)) {
        cout << "\nRoom already booked!\n";
        return;
    }

    ofstream file("records.dat", ios::app);
    cin.ignore();

    cout << "Enter Customer Name: ";
    char name[50];
    cin.getline(name, 50);

    int days;
    cout << "Enter Days: ";
    cin >> days;

    float bill = days * 1000;

    // Assign values
    h = Hotel();
    // Using hack since private fields
    // Better version: use constructor (can upgrade if needed)

    // Temporary workaround
    *((int*)((char*)&h)) = room;
    strcpy((char*)&h + sizeof(int), name);
    *((int*)((char*)&h + sizeof(int) + 50)) = days;
    *((float*)((char*)&h + sizeof(int) + 50 + sizeof(int))) = bill;

    file.write((char*)&h, sizeof(h));
    file.close();

    cout << "\nRoom Booked Successfully!\n";
}

// 📋 VIEW BOOKINGS
void viewBookings() {
    Hotel h;
    ifstream file("records.dat");

    if (!file) {
        cout << "\nNo bookings found!\n";
        return;
    }

    while (file.read((char*)&h, sizeof(h))) {
        h.display();
        cout << "------------------------\n";
    }

    file.close();
}

// ❌ CHECKOUT (DELETE)
void checkout() {
    int room;
    cout << "\nEnter Room Number for Checkout: ";
    cin >> room;

    Hotel h;
    ifstream file("records.dat");
    ofstream temp("temp.dat");

    bool found = false;

    while (file.read((char*)&h, sizeof(h))) {
        if (h.getRoomNo() == room) {
            found = true;
            cout << "\nCheckout Details:\n";
            h.display();
            continue;
        }
        temp.write((char*)&h, sizeof(h));
    }

    file.close();
    temp.close();

    remove("records.dat");
    rename("temp.dat", "records.dat");

    if (found)
        cout << "\nCheckout Successful!\n";
    else
        cout << "\nRoom Not Found!\n";
}

// 🏁 MAIN FUNCTION
int main() {
    if (!login())
        return 0;

    int choice;

    do {
        cout << "\n\n--- HOTEL MANAGEMENT SYSTEM ---\n";
        cout << "1. Book Room\n";
        cout << "2. View Bookings\n";
        cout << "3. Checkout\n";
        cout << "4. Exit\n";

        cout << "Enter Choice: ";
        cin >> choice;

        switch (choice) {
            case 1: addBooking(); break;
            case 2: viewBookings(); break;
            case 3: checkout(); break;
            case 4: cout << "\nThank You!\n"; break;
            default: cout << "\nInvalid Choice!\n";
        }

    } while (choice != 4);

    return 0;
}
