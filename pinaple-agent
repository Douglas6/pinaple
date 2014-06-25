#!/usr/bin/env python

# Author: Douglas Otwell
# This software is released to the public domain

# The Software is provided "as is" without warranty of any kind, either express or implied, 
# including without limitation any implied warranties of condition, uninterrupted use, 
# merchantability, fitness for a particular purpose, or non-infringement.

import os
import sys
import dbus
import dbus.service
import dbus.mainloop.glib
import gobject
from optparse import OptionParser

pin_code = "0000"

class Agent(dbus.service.Object):

    @dbus.service.method("org.bluez.Agent", in_signature="o", out_signature="s")
    def RequestPinCode(self, device_path):
        device = dbus.Interface(bus.get_object("org.bluez", device_path), "org.bluez.Device")
        properties = device.GetProperties()
        print "Pairing and trusting device %s [%s]" % (properties["Alias"], properties["Address"])
        device.SetProperty("Trusted", dbus.Boolean(True))
        return pin_code

    @dbus.service.method("org.bluez.Agent", in_signature="ou", out_signature="")
    def RequestConfirmation(self, device_path, passkey):
        device = dbus.Interface(bus.get_object("org.bluez", device_path), "org.bluez.Device")
        properties = device.GetProperties()
        print "Pairing and trusting device %s [%s] with passkey [%s]" % (properties["Alias"], properties["Address"], passkey)
        device.SetProperty("Trusted", dbus.Boolean(True))
        return

    @dbus.service.method("org.bluez.Agent", in_signature="os", out_signature="")
    def Authorize(self, device, uuid):
        print "Authorize (%s, %s)" % (device, uuid)
        authorize = raw_input("Authorize connection (yes/no): ")
        if (authorize == "yes"):
            return
        raise Rejected("Connection rejected by user")

if __name__ == '__main__':

    if (os.getuid() != 0):
        print "You must have root privileges to run this agent. Try 'sudo pinaple-agent [--pin <PIN>]'"
        raise SystemExit

    parser = OptionParser()
    parser.add_option("-p", "--pin", action="store", dest="pin_code", help="PIN code to pair with", metavar="PIN")
    (options, args) = parser.parse_args()

    # use the pin code if provided
    if (options.pin_code):
        pin_code = options.pin_code

    # get the dbus system bus
    dbus.mainloop.glib.DBusGMainLoop(set_as_default=True)
    bus = dbus.SystemBus()

    # get the Bluez manager and default bluetooth adapter
    manager = dbus.Interface(bus.get_object("org.bluez", "/"), "org.bluez.Manager")
    adapter_path = manager.DefaultAdapter()
    adapter = dbus.Interface(bus.get_object("org.bluez", adapter_path), "org.bluez.Adapter")

    # set the adapter to discoverable 
    print "Making Bluetooth adapter discoverable"
    adapter.SetProperty("Discoverable", dbus.Boolean(True))

    # register the pinaple agent
    agent_path = "/pinaple/agent"
    agent = Agent(bus, agent_path)
    try:
        adapter.RegisterAgent(agent_path, "KeyboardDisplay")
        print "Waiting to pair devices...."
    except dbus.exceptions.DBusException, ex:
        print "A Bluetooth agent is currently running. Exiting pinaple-agent", ex
        raise SystemExit

    mainloop = gobject.MainLoop()
    try:
        mainloop.run()
    except KeyboardInterrupt:
        print "\nMaking Bluetooth adapter undiscoverable"
        adapter.SetProperty("Discoverable", dbus.Boolean(False, variant_level=1))
        adapter.UnregisterAgent(agent_path)
#        raise SystemExit
        mainloop.quit()
