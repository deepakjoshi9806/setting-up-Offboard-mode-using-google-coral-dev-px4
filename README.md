# setting-up-Offboard-mode-using-google-coral-dev-px4
#### the below setup is for pixhawk cube with px4 as firmware,  google coral dev board as onboard computer. this is a standard method and will more or  less work for any ARM based boards.
#### Offboard mode is setup using MAVSDK here. the steps are as follows:
- ssh into the board and install the mavsdk package using pip3
`pip3 install mavsdk`
> the above method will install all the required packages along with the mavsdk server
- start a mavsdk server with 55555 (say) as port number and **/dev/serial/by-id/usb-CubePilot_CubeOrange_0-if00** as **serial** port
`/usr/local/lib/python3.7/dist-packages/mavsdk/bin/mavsdk_server -p 55555 serial:///dev/serial/by-id/usb-CubePilot_CubeOrange_0-if00 `
> here `/dev/serial/by-id/usb-CubePilot_CubeOrange_0-if00` is obtained after connecting pixhawk and google coral board using USB cable. 
- ones the mavsdk server is up and running, write a mavsdk python script to test it
- change the following lines in the code as follows:
`drone = System(mavsdk_server_address="localhost", port=55555)`
`await drone.connect()` 
> only these lines have to be changed,  rest all can be altered based on requirements
#### offboard mode 
- ones the server is up run the below  scripts
> this script enables offboard mode 


      import asyncio
    
	  from mavsdk import System
      from mavsdk.offboard import (OffboardError, PositionNedYaw)
    
      async def run():
      """ Does Offboard control using position NED coordinates. """
    
        drone = System(mavsdk_server_address="localhost", port=55555)
        await drone.connect()
    
        print("Waiting for drone to connect...")
        async for state in drone.core.connection_state():
            if state.is_connected:
                print(f"-- Connected to drone!")
                break
    
        print("-- Arming")
        await drone.action.arm()
    
        print("-- Setting initial setpoint")
        await drone.offboard.set_position_ned(PositionNedYaw(0.0, 0.0, 0.0, 0.0))
    
        print("-- Starting offboard")
        try:
            await drone.offboard.start()
        except OffboardError as error:
            print(f"Starting offboard mode failed \
                    with error code: {error._result.result}")
            print("-- Disarming")
            await drone.action.disarm()
            return
    
        print("-- Go 0m North, 0m East, -1.0m Down \
                within local coordinate system")
        await drone.offboard.set_position_ned(
                PositionNedYaw(0.0, 0.0, -1.0, 0.0))
    
        print("-- Stopping offboard")
        try:
            await drone.offboard.stop()
        except OffboardError as error:
            print(f"Stopping offboard mode failed \
                    with error code: {error._result.result}")
     
      if __name__ == "__main__":
          # Run the asyncio loop
          asyncio.run(run())
