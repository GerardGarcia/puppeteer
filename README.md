# puppeteer
Performance analysis library


# Puppeteer

Ppuppeteer can be used to study the behavior and analyze the performance of any *nix application. Puppeteer is based on the [Dyninst](http://www.dyninst.org/sites/default/files/manuals/dyninst/DyninstAPI.pdf) library and is implemented in C++11.

Puppeteer is composed of a set of libraries that facilitate the insertion of hooks at different points of the analyzed program, which we call a puppet, that transmit information about it back to the analyzing program, which we call the puppet master.

The main strength of this tool is that all hooks are inserted at runtime, therefore the code of the puppet does not need any modification, it just need to be compiled with debug symbols.

# Design

![alt text](https://raw.githubusercontent.com/GerardGarcia/puppeteer/master/img/puppeteer_design.png "Puppeteer design")

Overview:

 * The puppet master creates a shared memory segment (SHM) between the puppet and the puppet master. The events hooked to the puppet use this segment to store information about the state of the puppet so that this information could be accessed from the puppet master.
 * Then, it loads the puppetLib into the puppet process. In this lib we find the events that will be executed by the puppet and a wrapper for each event that provides synchronization access to the shared memory.
 * Finally, the puppet master gets the event from the shared memory and adds it to the event list contained in the puppet master lib.


# puppetLib

The puppetLib library is linked at runtime to the puppet and contains the events that can then be hooked to it. These are the available events:

| function                                                        | id          
| :-------------------------------------------------------------- |:-----------------------|
| static int puppeteerEventSimple(uint8_t data[MAX_EVENT_DATA]);  | puppeteerEventSimpleId|

 * puppeteerEventSimpleId: Registers when the event has been triggered (with nanosecond granularity) together with the supplied data. This data identifies the event in the event list.

# puppetMasterLib

The puppetMasterLib realize all the interaction with the puppet. All its interface is contained in one class:

```
class puppeteerCtx
{
public:
	puppeteerCtx();
	~puppeteerCtx();
	void addPuppet(const string puppetName, const string puppetCmd);
	void addEvent(  const string puppetName,
	                const string function,
	                const bool multiple,
	                const puppeteerEventLoc loc,
	                const puppeteerEvent_e eventId,
	                const char data[MAX_EVENT_DATA]   );
	void addAction(const int delay, const function<void()> a);
	void startTest(const int secs, const bool endTest, const bool forceEnd);
	void endTest(const int secs, const bool force);
	vector<puppeteerEvent_t> getEvents(const string puppetName);
private:
	class impl;
	unique_ptr<impl> pimpl;
};
```


