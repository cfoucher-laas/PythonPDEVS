Tracers
=======

PythonPDEVS provides several features that allow you to trace the execution of your model.
Here, we present four built-in tracers, and present the interface used to create your own tracer.
For all tracers, providing the *None* object as the filename causes the trace to be printed to stdout, instead of writing it to a file.

In the context of distributed simulation, tracing is a *destructive event*, meaning that it can only occur after the GVT.
As such, tracing happens in blocks as soon as fossil collection occurs.

Verbose
-------

The first tracer is the verbose tracer, which generates a purely textual trace of your model execution.
It has already been presented before, and was used by invoking the following configuration option::

    sim.setVerbose(None)

Note that this is the only built-in tracer that works without additional configuration of the model.
An example snippet is shown below::

    __  Current Time:       0.00 __________________________________________


            INITIAL CONDITIONS in model <trafficSystem.trafficLight>
                    Initial State: red
                    Next scheduled internal transition at time 58.50


            INITIAL CONDITIONS in model <trafficSystem.policeman>
                    Initial State: idle
                    Next scheduled internal transition at time 200.00


    __  Current Time:      58.50 __________________________________________


            INTERNAL TRANSITION in model <trafficSystem.trafficLight>
                    New State: green
                    Output Port Configuration:
                            port <OBSERVED>:
                                    grey
                    Next scheduled internal transition at time 108.50


    __  Current Time:     108.50 __________________________________________


            INTERNAL TRANSITION in model <trafficSystem.trafficLight>
                    New State: yellow
                    Output Port Configuration:
                            port <OBSERVED>:
                                    yellow
                    Next scheduled internal transition at time 118.50

    ...

    __  Current Time:     200.00 __________________________________________ 


        EXTERNAL TRANSITION in model <trafficSystem.trafficLight>
                Input Port Configuration:
                        port <INTERRUPT>:
                                toManual
                New State: manual
                Next scheduled internal transition at time inf


        INTERNAL TRANSITION in model <trafficSystem.policeman>
                New State: working
                Output Port Configuration:
                        port <OUT>:
                                toManual
                Next scheduled internal transition at time 300.00

XML
---

The second tracer is the XML tracer, which generates an XML-structured trace of your model execution, compliant to the notation presented in `Bill Song's thesis <http://msdl.cs.mcgill.ca/people/bill/thesis/latexthesis.pdf>`_.
It can also simply be enabled by setting the following configuration option::

    sim.setXML(None)

To enable this XML tracing, the model has to be augmented with facilities to dump the current state in the form of an XML notation.
This is done with the *toXML()* method, which has to be defined for all complex states, and must return a string to be embedded in the XML.
This string is pasted as-is in the trace file, and should therefore not contain forbidden characters (e.g., <).
For example, a *toXML* method for the traffic light can look as follows::

    class TrafficLightMode:
        ...

        def toXML(self):
            return "<mode>%s</mode>" % self.__colour

An example snippet is shown below::

    <trace>
        <event>
            <model>trafficSystem.trafficLight</model>
            <time>0.0</time>
            <kind>EX</kind>
            <state>
                <mode>red</mode><![CDATA[red]]>
            </state>
        </event>
        <event>
            <model>trafficSystem.policeman</model>
            <time>0.0</time>
            <kind>EX</kind>
            <state>
                <mode>idle</mode><![CDATA[idle]]>
            </state>
        </event>
        <event>
            <model>trafficSystem.trafficLight</model>
            <time>58.5</time>
            <kind>IN</kind>
            <port name="OBSERVED" category="O">
                <message>grey</message>
            </port>
            <state>
                <mode>green</mode><![CDATA[green]]>
            </state>
        </event>
        ...
        <event>
            <model>trafficSystem.policeman</model>
            <time>200.0</time>
            <kind>IN</kind>
            <port name="OUT" category="O">
                <message>toManual</message>
            </port>
            <state>
                <mode>working</mode><![CDATA[working]]>
            </state>
        </event>
        <event>
            <model>trafficSystem.trafficLight</model>
            <time>200.0</time>
            <kind>EX</kind>
            <port name="INTERRUPT" category="I">
                <message>toManual</message>
            </port>
            <state>
                <mode>manual</mode><![CDATA[manual]]>
            </state>
        </event>
    </trace>


VCD
---

TODO

Cell
----

The cell tracer is discussed separately, as it has very specific behaviour and is only applicable to a select number of models.
