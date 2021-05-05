Functional requirements: Hotel/Restro onboarding, hotel/restro manager can see bookings, User can search & filter hotel/restro for given facility/cuisine along with location and datetime, user can book/cancel/update room/table for specific capacity for specific datetime slot, charge users for booking to reduce no-shows and refund same post billing

Non functional requirements: Low latency, High availability, Use transactions to lock rows before updating (No >1 booking for a slot for same datetime slot) 

Scale: 500k hotels/restro, 10M rooms/tables, 

Services: User service, Hotel/restro service, Booking service, search service, payment service, notification service, Booking Management Service

HLD
----

(a) User onboarding and profile data flow from user app -> LB1 -> Userservice (read from cache first, write int db first) -> userdata_cache (redis cluster) -> userdb (write master- read slaves cluster)

(b) hotel/restro onboarding and profile data flow from hotel/restro app -> LB2 -> H/R_service (read from cache first, write int db first, once written push the data to message broker as producer) -> H/R_data_cache (redis cluster) -> H/R_db (write master- read slaves cluster) -> message broker (Kafka topic: new_hotel) 

(c) hotel search flow from user app -> LB3 -> Search Service -> searchdata_cache (redis cluster) -> searchdb (elastic search cluster) -> search consumer pulls new hotels and updated booking data from message broker -> message broker (kafka topic: new_hotel, new_booking)

(d) booking confirmation flow from user app -> LB4 -> Booking Service (saves booking data in DB and send sync requests waiting for payment to finish, once success, marking booking confirm in bookingdata_db and push this data into message broker)-> Payment Service (comfirms payment success/failure to booking service)-> bookingdata_cache (redis cluster) -> bookingdata_db (write master- read slaves cluster) -> message broker (Kafka topic: new_booking) -> notification service (consumes new bookng data and send notification to users)

(e) booking cancelled/availed flow from user app -> LB4 -> Booking Service (marks booking data as cancelled /availed in DB and push this data to message broker as producer) -> bookingdata_cache (redis cluster) -> bookingdata_db (write master- read slaves cluster) -> message broker (Kafka topic: booking_availed, booking_cancelled) -> Archive Service (consumes availed/cancelled booking data and pushes this to archive_db)-> archive_db (cassandra cluster)

(f) see current and past bookings data flow from user and hotel/restro both apps -> LB5 -> Booking Management service (read from bookingdata_cache for current booking and read from archive_db for past bookings) 