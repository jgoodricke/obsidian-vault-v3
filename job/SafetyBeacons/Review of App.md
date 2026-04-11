  - [P1] Authorize the payment before accepting the driver — /home/james/Projects/
    safetyBeacons/find_my_ride_final/js/payment.js:121-124
    confirmPayment() never calls authorizePayment() or /api/payment.php; after only client-
    side Luhn/expiry/CVC checks it immediately accepts the bid and later marks
    paymentCompleted true. In any non-dev deployment this means a syntactically valid card
    number can book a ride with no authorization at all, and currentPaymentId stays null so
    there is nothing to capture when the trip finishes.
  - [P1] Verify ride ownership before accepting a bid — /home/james/Projects/safetyBeacons/
    find_my_ride_final/api/rider/rides.php:95-103
    This branch validates the bid, but unlike cancel, complete, and rate_driver it never
    checks that the caller owns the ride. Nearby drivers already receive each ride_id from
    get_available_rides.php and know their own bid amount, so a modified driver client can
    POST action=accept here and force the ride into accepted without the rider ever
    confirming in the payment flow.
  - [P1] Enforce the driver session on state-changing APIs — /home/james/Projects/
    safetyBeacons/find_my_ride_final/api/driver/location.php:20-25
    The handler trusts driver_id from JSON and never checks $_SESSION['driver_id'] or the
    session token minted in auth.php. Any caller can therefore impersonate another driver
    and overwrite their live GPS point; if that driver has an accepted ride, /api/rider/
    tracking.php will immediately start showing the spoofed coordinates to the rider. The
    same trust pattern is used in the other state-changing driver endpoints as well.
  - [P2] Reject zero or negative bid amounts — /home/james/Projects/safetyBeacons/
    find_my_ride_final/api/driver/bids.php:59-63
    There is no lower-bound validation on bid_amount here. A driver can submit 0 or a
    negative custom bid, it will sort to the top of the rider's bid list as the cheapest
    option, and accepting it records a non-positive fare that the payment API later refuses
    to authorize (amount <= 0). That makes the bidding flow easy to break with a single
    malformed bid.


It may be worth starting with an app first for drivers. I don't know if using a web browser is practical for this sort of thing.

This needs a framework on the backend.