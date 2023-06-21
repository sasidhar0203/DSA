# DSA
#include <iostream>
#include <unordered_map>
#include <stack>
#include <queue>
#include <list>
#include <algorithm>

using namespace std;

struct Event {
    string name;
    string date;
    string time;
    list<string> attendees;
};

unordered_map<string, Event> events;
stack<Event> undoStack;
queue<Event> waitingList;

void addEvent() {
    Event event;
    cout << "Enter event name: ";
    cin.ignore(); // Ignore any remaining newline character in the input stream
    getline(cin, event.name);
    cout << "Enter event date: ";
    cin >> event.date;
    cout << "Enter event time: ";
    cin >> event.time;

    bool conflict = false;
    for (const auto& pair : events) {
        const Event& existingEvent = pair.second;
        if (existingEvent.date == event.date) {
            conflict = true;
            break;
        }
    }

    if (conflict) {
        waitingList.push(event);
        cout << "Event added to waiting list due to date conflict!" << endl;
    } else {
        events[event.name] = event;
        undoStack.push(event);
        cout << "Event added successfully!" << endl;
    }
}

void removeEvent() {
    string eventName;
    cout << "Enter event name: ";
    cin.ignore();
    getline(cin, eventName);
    
    auto eventIter = events.find(eventName);
    if (eventIter != events.end()) {
        undoStack.push(eventIter->second); // Push the event onto the undo stack before erasing it
        
        // Check if there is an event in the waiting list with the same date
        bool foundWaitingEvent = false;
        queue<Event> tempQueue;
        while (!waitingList.empty()) {
            Event waitingEvent = waitingList.front();
            waitingList.pop();
            if (waitingEvent.date == eventIter->second.date) {
                foundWaitingEvent = true;
                events[waitingEvent.name] = waitingEvent; // Add waiting event to event list
                cout << "Event from waiting list added to main list!" << endl;
                cout << "Slot confirmed for the event in the waiting list." << endl;
                cout << "Event details:" << endl;
                cout << "Event Name: " << waitingEvent.name << endl;
                cout << "Event Date: " << waitingEvent.date << endl;
                cout << "Event Time: " << waitingEvent.time << endl;
                cout << "Attendees: ";
                for (const auto& attendee : waitingEvent.attendees) {
                    cout << attendee << " ";
                }
                cout << endl;
            } else {
                tempQueue.push(waitingEvent);
            }
        }
        waitingList = tempQueue; // Restore the remaining events in the waiting list
        
        events.erase(eventIter);
        cout << "Event removed successfully!" << endl;
        
        if (!foundWaitingEvent) {
            cout << "No event found in the waiting list with the same date." << endl;
        }
    } else {
        cout << "Event not found!" << endl;
    }
}

void addAttendee() {
    string eventName;
    cout << "Enter event name: ";
    cin.ignore();
    getline(cin, eventName);

    auto eventIter = events.find(eventName);
    if (eventIter != events.end()) {
        string attendeeName;
        cout << "Enter attendee name: ";
        getline(cin, attendeeName);
        eventIter->second.attendees.push_back(attendeeName);
        cout << "Attendee added successfully!" << endl;
    } else {
        cout << "Event not found." << endl;
    }
}

void removeAttendee() {
    string eventName;
    cout << "Enter event name: ";
    cin.ignore();
    getline(cin, eventName);

    auto eventIter = events.find(eventName);
    if (eventIter != events.end()) {
        string attendeeName;
        cout << "Enter attendee name: ";
        getline(cin, attendeeName);

        auto attendeeIter = find(eventIter->second.attendees.begin(), eventIter->second.attendees.end(), attendeeName);
        if (attendeeIter != eventIter->second.attendees.end()) {
            eventIter->second.attendees.erase(attendeeIter);
            cout << "Attendee removed successfully!" << endl;
        } else {
            cout << "Attendee not found." << endl;
        }
    } else {
        cout << "Event not found." << endl;
    }
}

void viewEventDetails() {
    string eventName;
    cout << "Enter event name: ";
    cin.ignore();
    getline(cin, eventName);

    auto eventIter = events.find(eventName);
    if (eventIter != events.end()) {
        cout << "Event Name: " << eventIter->second.name << endl;
        cout << "Event Date: " << eventIter->second.date << endl;
        cout << "Event Time: " << eventIter->second.time << endl;
        cout << "Attendees: ";
        for (const auto& attendee : eventIter->second.attendees) {
            cout << attendee << " ";
        }
        cout << endl;
    } else {
        cout << "Event not found." << endl;
    }
}

void viewAllEvents() {
    if (events.empty()) {
        cout << "No events found!" << endl;
    } else {
        cout << "Events:" << endl;
        for (const auto& pair : events) {
            const Event& event = pair.second;
            cout << "Event Name: " << event.name << endl;
            cout << "Event Date: " << event.date << endl;
            cout << "Event Time: " << event.time << endl;
            cout << "Attendees: ";
            for (const auto& attendee : event.attendees) {
                cout << attendee << " ";
            }
            cout << endl;
        }
    }
}

void viewWaitingList() {
    if (waitingList.empty()) {
        cout << "Waiting list is empty!" << endl;
    } else {
        cout << "Waiting List:" << endl;
        int i = 1;
        queue<Event> tempQueue = waitingList;
        while (!tempQueue.empty()) {
            const Event& event = tempQueue.front();
            cout << i << ". Event Name: " << event.name << endl;
            cout << "   Event Date: " << event.date << endl;
            cout << "   Event Time: " << event.time << endl;
            cout << "   Attendees: ";
            for (const auto& attendee : event.attendees) {
                cout << attendee << " ";
            }
            cout << endl;
            tempQueue.pop();
            i++;
        }
    }
}

void undoLastOperation() {
    if (undoStack.empty()) {
        cout << "Nothing to undo!" << endl;
    } else {
        Event lastEvent = undoStack.top();
        undoStack.pop();
        auto it = events.find(lastEvent.name);
        if (it != events.end()) {
            events[lastEvent.name] = lastEvent;
        } else {
            events[lastEvent.name] = lastEvent;
        }
        cout << "Last operation undone successfully!" << endl;
    }
}

int main() {
    int choice;
    do {
        cout << "Event Management System" << endl;
        cout << "1. Add Event" << endl;
        cout << "2. Remove Event" << endl;
        cout << "3. Add Attendee" << endl;
        cout << "4. Remove Attendee" << endl;
        cout << "5. View Event Details" << endl;
        cout << "6. View All Events" << endl;
        cout << "7. View Waiting List" << endl;
        cout << "8. Undo Last Operation" << endl;
        cout << "9. Exit" << endl;
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice) {
            case 1:
                addEvent();
                break;
            case 2:
                removeEvent();
                break;
            case 3:
                addAttendee();
                break;
            case 4:
                removeAttendee();
                break;
            case 5:
                viewEventDetails();
                break;
            case 6:
                viewAllEvents();
                break;
            case 7:
                viewWaitingList();
                break;
            case 8:
                undoLastOperation();
                break;
            case 9:
                cout << "Exiting..." << endl;
                break;
            default:
                cout << "Invalid choice! Please try again." << endl;
                break;
        }
    } while (choice != 9);

    return 0;
}
