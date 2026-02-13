#include <iostream>
#include <string>
#include <unordered_set>
#include <unordered_map>
#include <queue>
#include <cstdlib>
#include <ctime>
#include <cctype>

using namespace std;

/* ---------- Utility Function ---------- */
void printSuccess(const string& action) {
    cout << "\n[ SUCCESS ] " << action << " completed successfully.\n";
}

/* ---------- Train Structure ---------- */
struct Train {
    int trainNo;
    string trainName;
    string source, destination;
    string startTime, endTime;
    float price;
    int count;

    Train() : count(0) {}
};

/* ---------- Passenger Linked List Node ---------- */
struct PassNode {
    int Reg_no;
    string fName, lName;
    int age;
    string gender;
    string train_class;
    PassNode* next;
    string date;
    int seatNo;

    PassNode() : next(nullptr) {}
};

/* ---------- Seat Matrix ---------- */
struct SeatMatrix {
    int seat[4][5];
    unordered_set<int> set;
    unordered_map<string, int> map;
    queue<PassNode*> waiting;

    SeatMatrix() {
        int k = 11;
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 5; j++) {
                seat[i][j] = k++;
            }
        }

        for (int i = 11; i <= 30; i++) {
            set.insert(i);
        }

        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 5; j++) {
                map[to_string(i) + "," + to_string(j)] = seat[i][j];
            }
        }
    }

    void display() {
        cout << "U\tM\tL\tL\tU\n";
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 5; j++) {
                cout << seat[i][j] << "\t";
            }
            cout << endl;
        }
    }
};

/* ---------- Booking Operations ---------- */
class Operations {
public:
    PassNode* book_ticket(PassNode* head, SeatMatrix& getSeat) {
        PassNode* newNode = new PassNode();

        while (true) {
            cout << "Enter first name: ";
            cin >> newNode->fName;
            bool valid = true;
            for (char c : newNode->fName)
                if (!isalpha(c)) valid = false;
            if (valid) break;
            cout << "Only alphabets allowed.\n";
        }

        while (true) {
            cout << "Enter last name: ";
            cin >> newNode->lName;
            bool valid = true;
            for (char c : newNode->lName)
                if (!isalpha(c)) valid = false;
            if (valid) break;
            cout << "Only alphabets allowed.\n";
        }

        cout << "Age: ";
        cin >> newNode->age;

        do {
            cout << "Gender (M/F): ";
            cin >> newNode->gender;
        } while (!(newNode->gender == "M" || newNode->gender == "F" ||
                   newNode->gender == "m" || newNode->gender == "f"));

        newNode->Reg_no = rand() % 9000 + 1000;

        cout << "Class:\n1. Sleeper\n2. 1st AC\n3. 2nd AC\n4. 3rd AC\n";
        int ch;
        cin >> ch;
        if (ch == 1) newNode->train_class = "Sleeper";
        else if (ch == 2) newNode->train_class = "1st AC";
        else if (ch == 3) newNode->train_class = "2nd AC";
        else if (ch == 4) newNode->train_class = "3rd AC";
        else newNode->train_class = "General";

        while (true) {
            cout << "Date (dd/mm/yyyy): ";
            cin >> newNode->date;
            if (newNode->date.length() == 10 &&
                newNode->date[2] == '/' &&
                newNode->date[5] == '/')
                break;
            cout << "Invalid date format.\n";
        }

        if (getSeat.set.empty()) {
            cout << "No seats available. Added to waiting list.\n";
            newNode->seatNo = 0;
            getSeat.waiting.push(newNode);
        } else {
            getSeat.display();
            int s;
            do {
                cout << "Enter seat number: ";
                cin >> s;
            } while (!getSeat.set.count(s));

            for (int i = 0; i < 4; i++)
                for (int j = 0; j < 5; j++)
                    if (getSeat.seat[i][j] == s)
                        getSeat.seat[i][j] = 0;

            getSeat.set.erase(s);
            newNode->seatNo = s;
        }

        cout << "\n--- Ticket ---\n";
        cout << "Name: " << newNode->fName << " " << newNode->lName << endl;
        cout << "Seat: " << newNode->seatNo << endl;
        cout << "Reg No: " << newNode->Reg_no << endl;

        if (!head) return newNode;
        PassNode* t = head;
        while (t->next) t = t->next;
        t->next = newNode;
        return head;
    }

    void display_passenger(PassNode* head) {
        if (!head) {
            cout << "No passengers.\n";
            return;
        }
        while (head) {
            cout << "Name: " << head->fName << " " << head->lName
                 << " | Seat: " << head->seatNo << endl;
            head = head->next;
        }
    }

    PassNode* cancelBooking(PassNode* head, SeatMatrix& q) {
        if (!head) {
            cout << "No bookings found.\n";
            return nullptr;
        }

        cout << "Enter seat number to cancel: ";
        int s;
        cin >> s;

        PassNode* cur = head;
        PassNode* prev = nullptr;

        while (cur && cur->seatNo != s) {
            prev = cur;
            cur = cur->next;
        }

        if (!cur) {
            cout << "Seat not found.\n";
            return head;
        }

        if (!prev) head = cur->next;
        else prev->next = cur->next;

        waitingToConfirm(s, q, head);

        delete cur;
        cout << "Booking cancelled.\n";
        return head;
    }

    void waitingToConfirm(int seatNumber, SeatMatrix& q, PassNode*& head) {
        if (q.waiting.empty()) {
            for (auto& entry : q.map) {
                if (entry.second == seatNumber) {
                    string idx = entry.first;
                    size_t pos = idx.find(',');
                    int i = stoi(idx.substr(0, pos));
                    int j = stoi(idx.substr(pos + 1));
                    q.seat[i][j] = seatNumber;
                    break;
                }
            }
            q.set.insert(seatNumber);
        } else {
            PassNode* wait = q.waiting.front();
            q.waiting.pop();
            wait->seatNo = seatNumber;

            for (int i = 0; i < 4; i++) {
                for (int j = 0; j < 5; j++) {
                    if (q.seat[i][j] == seatNumber) {
                        q.seat[i][j] = 0;
                        break;
                    }
                }
            }

            if (!head) {
                head = wait;
            } else {
                PassNode* ptr = head;
                while (ptr->next) ptr = ptr->next;
                ptr->next = wait;
            }
            cout << "Waiting passenger confirmed.\n";
        }
    }
};

/* ---------- Train Manager ---------- */
struct Operation2 {
    Train T[5];
    Operations obj;
    PassNode* heads[5];
    SeatMatrix seats[5];

    Operation2() {
        for (int i = 0; i < 5; i++) heads[i] = nullptr;

        T[0].trainNo = 101; T[0].trainName = "Rajdhani Express";
        T[1].trainNo = 102; T[1].trainName = "Shatabdi Express";
        T[2].trainNo = 103; T[2].trainName = "Duronto Express";
        T[3].trainNo = 104; T[3].trainName = "Garib Rath";
        T[4].trainNo = 105; T[4].trainName = "Humsafar Express";
    }

    void bookTrain() {
        for (int i = 0; i < 5; i++)
            cout << i + 1 << ". " << T[i].trainName << endl;

        int c;
        cin >> c;
        c--;
        if (c >= 0 && c < 5) {
            heads[c] = obj.book_ticket(heads[c], seats[c]);
            printSuccess("Train booking");
        }
    }

    void displayAllPassengers() {
        for (int i = 0; i < 5; i++) {
            cout << "\n" << T[i].trainName << ":\n";
            obj.display_passenger(heads[i]);
        }
        printSuccess("Passenger display");
    }

    void cancel() {
        for (int i = 0; i < 5; i++)
            cout << i + 1 << ". " << T[i].trainName << endl;

        int c;
        cin >> c;
        c--;
        if (c >= 0 && c < 5) {
            heads[c] = obj.cancelBooking(heads[c], seats[c]);
            printSuccess("Ticket cancellation");
        }
    }
};

/* ---------- Main ---------- */
int main() {
    srand(time(0));
    Operation2 op;

    while (true) {
        cout << "\n--- Railway Registration System ---\n";
        cout << "1. Book Train\n";
        cout << "2. Display All Passengers\n";
        cout << "3. Cancel Booking\n";
        cout << "4. Exit\n";
        cout << "Choice: ";

        int ch;
        cin >> ch;

        if (ch == 1) op.bookTrain();
        else if (ch == 2) op.displayAllPassengers();
        else if (ch == 3) op.cancel();
        else if (ch == 4) exit(0);
        else cout << "Invalid choice.\n";
    }
}
