# DS-Practicals

#

1st Practical

Server.java

import java.rmi.Naming;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.server.UnicastRemoteObject;
import java.util.ArrayList;

public class Server implements Service {
    private ArrayList<String> messages = new ArrayList<>();

    public Server() throws RemoteException {
        UnicastRemoteObject.exportObject(this, 0);
    }

    public void receiveMessage(String message) throws RemoteException {
        System.out.println("Received message: " + message);
        messages.add(message);
    }

    public static void main(String[] args) {
        try {
            Server server = new Server();
            Registry registry = LocateRegistry.createRegistry(1099);
            Naming.rebind("rmi://localhost/Service", server);
            System.out.println("Server ready");
        } catch (Exception e) {
            System.out.println("Server exception: " + e.toString());
            e.printStackTrace();
        }
    }
}

interface Service extends java.rmi.Remote {
    void receiveMessage(String message) throws RemoteException;
}

Client.java

import java.rmi.Naming;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.util.Scanner;

public class Client implements Runnable {
    private Service service;

    public Client(Service service) {
        this.service = service;
    }

    public void run() {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.print("Enter message: ");
            String message = scanner.nextLine();
            try {
                service.receiveMessage(message);
            } catch (RemoteException e) {
                System.out.println("Client exception: " + e.toString());
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        try {
            Registry registry = LocateRegistry.getRegistry(1099);
            Service service = (Service) Naming.lookup("rmi://localhost/Service");
            Client client = new Client(service);
            Thread thread = new Thread(client);
            thread.start();
        } catch (Exception e) {
            System.out.println("Client exception: " + e.toString());
            e.printStackTrace();
        }
    }
}

















4th Practical

server.py

# Python3 program imitating a clock server
from functools import reduce
from dateutil import parser
import threading
import datetime
import socket
import time

# datastructure used to store client address and clock data
client_data = {}
''' nested thread function used to receive
	clock time from a connected client '''

def startReceivingClockTime(connector, address):
    while True:
        # receive clock time
        clock_time_string = connector.recv(1024).decode()
        clock_time = parser.parse(clock_time_string)
        clock_time_diff = datetime.datetime.now() - \
            clock_time

        client_data[address] = {
            "clock_time": clock_time,
            "time_difference": clock_time_diff,
            "connector": connector
        }
        print("Client Data updated with: " + str(address),
              end="\n\n")
        time.sleep(5)

''' master thread function used to open portal for
	accepting clients over given port '''
def startConnecting(master_server):
    # fetch clock time at slaves / clients
    while True:
        # accepting a client / slave clock client
        master_slave_connector, addr = master_server.accept()
        slave_address = str(addr[0]) + ":" + str(addr[1])
        print(slave_address + " got connected successfully")
        current_thread = threading.Thread(
            target=startReceivingClockTime,
            args=(master_slave_connector,
                  slave_address, ))
        current_thread.start()

# subroutine function used to fetch average clock difference
def getAverageClockDiff():
    current_client_data = client_data.copy()
    time_difference_list = list(client['time_difference']
                                for client_addr, client
                                in client_data.items())

    sum_of_clock_difference = sum(time_difference_list,
                                  datetime.timedelta(0, 0))

    average_clock_difference = sum_of_clock_difference \
        / len(client_data)

    return average_clock_difference

''' master sync thread function used to generate
	cycles of clock synchronization in the network '''
def synchronizeAllClocks():
    while True:
        print("New synchronization cycle started.")
        print("Number of clients to be synchronized: " +
              str(len(client_data)))
        if len(client_data) > 0:
            average_clock_difference = getAverageClockDiff()
            for client_addr, client in client_data.items():
                try:
                    synchronized_time = \
                        datetime.datetime.now() + \
                        average_clock_difference
                    client['connector'].send(str(
                        synchronized_time).encode())
                except Exception as e:
                    print("Something went wrong while " +
                          "sending synchronized time " +
                          "through " + str(client_addr))
        else:
            print("No client data." +
                  " Synchronization not applicable.")
        print("\n\n")
        time.sleep(5)

# function used to initiate the Clock Server / Master Node
def initiateClockServer(port=8080):
    master_server = socket.socket()
    master_server.setsockopt(socket.SOL_SOCKET,
                             socket.SO_REUSEADDR, 1)
    print("Socket at master node created successfully\n")
    master_server.bind(('', port))
    # Start listening to requests
    master_server.listen(10)
    print("Clock server started...\n")
    # start making connections
    print("Starting to make connections...\n")
    master_thread = threading.Thread(
        target=startConnecting,
        args=(master_server, ))
    master_thread.start()
    # start synchronization
    print("Starting synchronization parallelly...\n")
    sync_thread = threading.Thread(
        target=synchronizeAllClocks,
        args=())
    sync_thread.start()

# Driver function
if __name__ == '__main__':
    # Trigger the Clock Server
    initiateClockServer(port=8080)



client.py

# Python3 program imitating a client process
from timeit import default_timer as timer
from dateutil import parser
import threading
import datetime
import socket
import time

# client thread function used to send time at client side
def startSendingTime(slave_client):
    while True:
        # provide server with clock time at the client
        slave_client.send(str(
            datetime.datetime.now()).encode())
        print("Recent time sent successfully",
              end="\n\n")
        time.sleep(5)

# client thread function used to receive synchronized time
def startReceivingTime(slave_client):
    while True:
        # receive data from the server
        Synchronized_time = parser.parse(
            slave_client.recv(1024).decode())
        print("Synchronized time at the client is: " +
              str(Synchronized_time),
              end="\n\n")

# function used to Synchronize client process time
def initiateSlaveClient(port=8080):
    slave_client = socket.socket()
    # connect to the clock server on local computer
    slave_client.connect(('127.0.0.1', port))
    # start sending time to server
    print("Starting to receive time from server\n")
    send_time_thread = threading.Thread(
        target=startSendingTime,
        args=(slave_client, ))
    send_time_thread.start()
    # start receiving synchronized from server
    print("Starting to receiving " +
          "synchronized time from server\n")
    receive_time_thread = threading.Thread(
        target=startReceivingTime,
        args=(slave_client, ))
    receive_time_thread.start()

# Driver function
if __name__ == '__main__':
    # initialize the Slave / Client
    initiateSlaveClient(port=8080)








5th Practical

ring-token.c

#include<stdio.h>
#include<conio.h>
#include<dos.h>
#include<time.h>

int main(){
    int  cs=0,pro=0;
    double run=5;
    char key='a';
    time_t t1,t2;
    printf("Press a key(except q) to enter a process into critical section.");
    printf(" \n Press q at any time to exit.");
    t1 = time(NULL) - 5;
    while(key!='q')
    {
        while(!kbhit())
        if(cs!=0)
        {
            t2 = time(NULL);
            if(t2-t1 > run)
            {
                printf("Process%d ",pro-1);
                printf(" exits critical section.\n");
                cs=0;
            }
        }
        key = getch();
        if(key!='q')
        {
            if(cs!=0)
            printf("Error: Another process is currently executing critical section Please wait till its execution is over.\n");
            else
            {
                printf("Process %d ",pro);
                printf(" entered critical section\n");
                cs=1;
                pro++;
                t1 = time(NULL);
            }
        }
    }
}







6th Practical

bully_ring.py

# we define MAX as the maximum number of processes our program can simulate
# we declare pStatus to store the process status; 0 for dead and 1 for alive
# we declare n as the number of processes
# we declare coordinator to store the winner of election

MAX = 20
pStatus = [0 for _ in range(MAX)]
n = 0
coordinator = 0

# def take_input():
#     global coordinator,n
#     n = int(input("Enter number of processes: "))
#     for i in range(1, n+1):
#         print("Enter Process ",i, " is alive or not(0/1): ")
#         x = int(input())
#         pStatus[i] = x
#         if pStatus[i]:
#             coordinator = i

def bully():
    " bully election implementation"
    global coordinator
    condition = True
    while condition:
        print('---------------------------------------------')
        print("1.CRASH\n2.ACTIVATE\n3.DISPLAY\n4.EXIT")
        print('---------------------------------------------\n')
        print("Enter your choice: ", end='')
        schoice = int(input())

        if schoice == 1:
            # we manually crash the process to see if our implementation
            # can elect another leader
            print("Enter process to crash: ", end='')
            crash = int(input())
            # if the process is alive then set its status to dead
            if (pStatus[crash] != 0):
                pStatus[crash] = 0
            else:
                print('Process', crash, ' is already dead!\n')
                break
            condition = True
            while condition:
                # enter another process to initiate the election
                print("Enter election generator id: ", end='')
                gid = int(input())
                if (gid == coordinator or pStatus[gid] == 0):
                    print("Enter a valid generator id!")
                condition = (gid == coordinator or pStatus[gid] == 0)
            flag = 0
            # if the coordinator has crashed then we need to find another leader
            if (crash == coordinator):
                # the election generator process will send the message to all higher process
                i = gid + 1
                while i <= n:
                    print("Message is  sent from", gid, " to", i, end='\n')
                    # if the higher process is alive then it will respond
                    if (pStatus[i] != 0):
                        subcoordinator = i
                        print("Response is sent from", i, " to", gid, end='\n')
                        flag = 1
                    i += 1
                # the highest responding process is selected as the leader
                if (flag == 1):
                    coordinator = subcoordinator
                # else if no higher process are alive then the election generator process
                # is selected as leader
                else:
                    coordinator = gid
            display()

        elif schoice == 2:
            # enter process to revive
            print("Enter Process ID to be activated: ", end='')
            activate = int(input())
            # if the entered process was dead then it is revived
            if (pStatus[activate] == 0):
                pStatus[activate] = 1
            else:
                print("Process", activate, " is already alive!", end='\n')
                break
            # if the highest process is activated then it is the leader
            if (activate == n):
                coordinator = n
                break
            flag = 0
            # else, the activated process sends message to all higher process
            i = activate + 1
            while i <= n:
                print("Message is  sent from", activate, "to", i, end='\n')
                # if higher process is active then it responds
                if (pStatus[i] != 0):
                    subcoordinator = i
                    print("Response is sent from", i,
                          "to", activate, end='\n')
                    flag = 1
                i += 1
            # the highest responding process is made the leader
            if flag == 1:
                coordinator = subcoordinator
            # if no higher process respond then the activated process is leader
            else:
                coordinator = activate
            display()

        elif schoice == 3:
            display()

        elif schoice == 4:
            pass

        condition = (schoice != 4)

def ring():
    " ring election implementation"
    global coordinator, n
    condition = True
    while condition:
        print('---------------------------------------------')
        print("1.CRASH\n2.ACTIVATE\n3.DISPLAY\n4.EXIT")
        print('---------------------------------------------\n')
        print("Enter your choice: ", end='')
        tchoice = int(input())
        if tchoice == 1:
            print("\nEnter process to crash : ", end='')
            crash = int(input())

            if pStatus[crash]:
                pStatus[crash] = 0
            else:
                print("Process", crash, "is already dead!", end='\n')
            condition = True
            while condition:
                print("Enter election generator id: ", end='')
                gid = int(input())
                if gid == coordinator:
                    print("Please, enter a valid generator id!", end='\n')
                condition = (gid == coordinator)

            if crash == coordinator:
                subcoordinator = 1
                i = 0
                while i < (n+1):
                    pid = (i + gid) % (n+1)
                    if pid != 0:     # since our process starts from 1 (to n)
                        if pStatus[pid] and subcoordinator < pid:
                            subcoordinator = pid
                        print("Election message passed from", pid, ": #Msg", subcoordinator, end='\n')
                    i += 1

                coordinator = subcoordinator
            display()

        elif tchoice == 2:
            print("Enter Process ID to be activated: ", end='')
            activate = int(input())
            if not pStatus[activate]:
                pStatus[activate] = 1
            else:
                print("Process", activate, "is already alive!", end='\n')
                break

            subcoordinator = activate
            i = 0
            while i < (n+1):
                pid = (i + activate) % (n+1)
                if pid != 0:    # since our process starts from 1 (to n)
                    if pStatus[pid] and subcoordinator < pid:
                        subcoordinator = pid
                    print("Election message passed from", pid,
                          ": #Msg", subcoordinator, end='\n')
                i += 1

            coordinator = subcoordinator
            display()

        elif tchoice == 3:
            display()

        condition = tchoice != 4


def choice():
    """ choice of options """
    while True:
        print('---------------------------------------------')
        print("1.BULLY ALGORITHM\n2.RING ALGORITHM\n3.DISPLAY\n4.EXIT")
        print('---------------------------------------------\n')
        fchoice = int(input("Enter your choice: "))

        if fchoice == 1:
            bully()
        elif fchoice == 2:
            ring()
        elif fchoice == 3:
            display()
        elif fchoice == 4:
            exit(0)
        else:
            print("Please, enter valid choice!")


def display():
    """ displays the processes, their status and the coordinator """
    global coordinator
    print('---------------------------------------------')
    print("PROCESS:", end='  ')
    for i in range(1, n+1):
        print(i, end='\t')
    print('\nALIVE:', end='    ')
    for i in range(1, n+1):
        print(pStatus[i], end='\t')
    print('\n---------------------------------------------')
    print('COORDINATOR IS', coordinator, end='\n')
    # print('----------------------------------------------')


if __name__ == '__main__':

    # take_input()

    n = int(input("Enter number of processes: "))
    for i in range(1, n+1):
        print("Enter Process ", i, " is alive or not(0/1): ")
        x = int(input())
        pStatus[i] = x
        if pStatus[i]:
            coordinator = i

    display()
    choice()

