# XMPP-Client-based-on-gloox
## Overview
Implement a xmpp client with gloox library based on xmpp stack protocal under Linux, supporting functions like “register”, “login”, “Add friend”, “send” and “chatting with friend”.

## Steps
- step 1: install gloox
  - command: sudo apt-get install libgloox-dev
- step 2: compile this client under linux
  - command: g++ -o myClient main.cpp -lgloox -lpthread -std=c++11


## Features
- Register
- Login
- Add a friend
- Show the contact list
- Select a buddy to send message
- Receive a message

### Client class
constructor of the myClientX class. class myClientX: public MessageHandler,  RegistrationHandler,  ConnectionListener
```C++
        myClientX(string userjid, string paswd) :
                        m_coreClient(0), m_reg(0)
        {
                //Constructor, register automatic ConnectionListener and Message Handler.
                JID jjid(userjid);
                m_coreClient = new Client(jjid, paswd);
                m_coreClient->registerMessageHandler(this);
                m_coreClient->registerConnectionListener(this);

                StringList calist;
                calist.push_back("/path/to/cacert.crt");
                m_coreClient->setCACerts(calist);
        }
  ``` 
### Register
Register a account to server, if register successfully will use this JID to login server automatically.
If registeration failed, app will exit. some servers limit the register interval to the same mac address, So you may register successfully for the first time, and fail next time, but you can register again successfully several hours later
```C++
int createAcc()
{
        /*
        *  id: the jid include name and server with format  user@server 
        *  name: the user name without @server 
        *  servername:  the server
        */
        string id, pwd, name, servername;
        cout << "****** register a new account ******" << endl;
        cout << " input your id with format user@server " << endl;
        cout << "-> ";
        cin >> id;
        cout << " input your password" << endl;
        cout << "-> ";
        cin >> pwd;

        size_t pos = id.find_first_of("@");  
        if (pos == std::string::npos)
        {
                cout << "wrong input of id, will exit now" << endl;
                return 0;
        }
        servername = id.substr(pos + 1, id.length() - pos - 1);   /*parse the input to get the server name*/
        name = id.substr(0, pos);        /*parse the input to get the user name*/
        myClientX b(servername, name, pwd);
        cout << "new account: " << id << " Registering ..... " << endl;
        b.createNewAcc();
        if(RegistrationSuccess == b.getRegRes())
                menu(id, pwd);
        else
                cout << "register failed, exit now" << endl;
}
```
### Process the result of registeration
when registeration function returns, process the registeration result
```C++
        virtual void handleRegistrationResult (const JID&,
                        RegistrationResult rs)
        {
                m_regRes = rs;
                switch (rs)
                {
                case RegistrationConflict:
                        cout << "Register result: User name already exists." << endl;
                        break;
                case RegistrationForbidden:
                        cout << "Register result: sender not sufficient permissions " << endl;
                        break;
                case RegistrationUnknownError:
                        cout << "Register result: a unknown error " << endl;
                        break;
                case RegistrationSuccess:
                        cout << "Register result: the register was successful" << endl;
                        break;
                }
                m_coreClient->disconnect();
        }
```        
### Add a friend
add a friend to contact list
```C++
        void addfriend (string id, string nick_fname)
        {
                StringList groups;
                string req("hi, i want to add you as my friend");
                m_coreClient->rosterManager()->subscribe(id, nick_fname, groups, req);
        }       

```        

### Get contact list
invoked when inputing a "buddy" command, receives the contack list of friends from server 
```C++
        void showBuddyList()
        {  //show all the friends
                Roster *roster = m_coreClient->rosterManager()->roster();
                cout << "buddy list" << endl;
                Roster::const_iterator iter = roster->begin(); // iteration to get all the buddy
                while(iter != roster->end())
                {
                        /*cout << "JID: " << setw(20) << (*iter).second->jidJID().full().c_str() <<
                         "  Name: " << setw(10) << (*iter).second->name().c_str() <<
                         (*iter).second->subscription() << endl;*/
                        cout << "JID: " << setw(20) << (*iter).second->jidJID().full().c_str() << endl;
                        ++iter;
                }
                cout << flush;
        }   

```    
### Receive a message
handleMessage: invoked when a new message comes from your buddy
```C++
virtual void handleMessage(const Message& msg, MessageSession* session =
                        0)
        {
                /* ouput the received message from server or buddy  */
                string strMsg = msg.body();
                timev = time(0); //show the received time
                struct tm * currenttm = localtime(&timev); //get the current time
                string time = asctime(currenttm);
                time = time.substr(0, time.size()-1);
                if (strMsg.size() > 0) // exclude the empty message string
                {
                        cout << time << "  " << msg.from().username() << " said: "<< strMsg
                                        << endl << "-> " << flush;
                }
        }
```
### Create a new Account
```C++
        void createNewAcc() 
        {
                m_coreClient = new Client(m_serv);
                m_coreClient->disableRoster();
                m_coreClient->registerConnectionListener(this);

                m_reg = new Registration(m_coreClient);
                m_reg->registerRegistrationHandler(this);

                m_coreClient->connect(true);
        }
```
### Send a Message
send a message to your buddy
```C++
        void sendMSG(string buddy, string strmsg)
        {
                //send message to a friend
                Message msg(Message::Chat, buddy, strmsg);
                m_coreClient->send(msg);
        }
```
