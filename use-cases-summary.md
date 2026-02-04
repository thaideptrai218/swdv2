# Consolidated Use Cases

## UC-01 - Register Account

**Source**: `docs/requirements/use-cases/UC-01-register-account.md`

**Use Case Name**: Register Account

**Use Case ID**: UC-01

**Summary**: Traveler creates new account by providing personal information and email. System validates information, creates account, and sends verification email to confirm registration.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A3: Traveler. Secondary: A5: Email Service

**Preconditions**: Traveler has internet access. System is operational. Traveler does not have existing account.

**Trigger**: Traveler clicks "Sign Up" or "Register" button on public portal

**Main Sequence**: 1. Traveler clicks "Sign Up" button<br>2. System displays registration form<br>3. Traveler enters personal information<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Traveler enters full name<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Traveler enters email address<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Traveler enters password<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. Traveler confirms password<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. Traveler accepts terms and conditions<br>4. Traveler submits registration form<br>5. System validates information format<br>6. System checks email availability<br>7. System creates user account<br>8. System sends verification email to traveler's email address via Email Service<br>9. System displays success message with instructions to verify email<br>10. Traveler opens verification email<br>11. Traveler clicks verification link<br>12. System validates verification token<br>13. System activates account<br>14. System displays account activation confirmation

**Alternative Sequences**: **Step 5**: If information format invalid (email format incorrect, password too weak), System displays specific error messages and returns to Step 2<br><br>**Step 6**: If email already registered, System displays "Email already exists" message and offers "Login" or "Reset Password" options<br><br>**Step 8**: If email delivery fails, System logs error and displays message to traveler to try again later or contact support<br><br>**Step 12**: If verification token expired (after 24 hours), System displays error message and offers option to resend verification email<br><br>**Step 12**: If verification token invalid, System displays error message and offers option to resend verification email

**Postconditions**: **Success**: Traveler account created and activated. Traveler can log in. Welcome email sent. Account status set to "Active".<br><br>**Failure**: No account created. Traveler informed of specific error.

**Outstanding Questions**: 1. Should system support social login (Google, Facebook) in addition to email registration?<br>2. What is the account lockout policy after multiple failed verification attempts?<br>3. Should system collect additional information during registration (phone number, nationality, date of birth)?<br>4. What is the data retention policy for unverified accounts?<br>5. Should system allow registration without email verification for immediate booking needs?

---

## UC-02 - Log In

**Source**: `docs/requirements/use-cases/UC-02-log-in.md`

**Use Case Name**: Log In

**Use Case ID**: UC-02

**Summary**: User (Traveler, Hostel Staff, or Platform Administrator) enters credentials to authenticate. System validates credentials and grants access to role-appropriate dashboard and features.

**Dependency**: Include: None. Extend: None. Included by: UC-05 (Book Accommodation), UC-07 (Submit Review)

**Actors**: Primary: A2a: Hostel Staff, A3: Traveler, A0: Platform Administrator. Secondary: None

**Preconditions**: User has registered account. Account is active (email verified). System is operational. User is not already logged in.

**Trigger**: User clicks "Log In" button or attempts to access protected feature requiring authentication

**Main Sequence**: 1. User clicks "Log In" button<br>2. System displays login form<br>3. User enters credentials<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. User enters email address<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. User enters password<br>4. User submits login form<br>5. System validates credentials<br>6. System verifies account status is active<br>7. System retrieves user role and permissions<br>8. System creates authenticated session<br>9. System redirects user to role-appropriate dashboard<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. If Traveler: redirect to traveler homepage<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. If Hostel Staff: redirect to staff dashboard<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. If Platform Administrator: redirect to admin dashboard<br>10. System displays welcome message with user name

**Alternative Sequences**: **Step 5**: If credentials invalid (email not found or password incorrect), System displays "Invalid email or password" error message, increments failed login counter, and returns to Step 2<br><br>**Step 5**: If failed login attempts exceed 5 within 15 minutes, System temporarily locks account for 30 minutes and displays lockout message with time remaining<br><br>**Step 6**: If account status is "Unverified", System displays message "Please verify your email address" and offers option to resend verification email<br><br>**Step 6**: If account status is "Suspended", System displays message "Account suspended. Contact support for assistance" and prevents login<br><br>**Step 6**: If account status is "Deleted", System displays "Invalid email or password" error (same as invalid credentials for security)<br><br>**Step 3**: User clicks "Forgot Password" link, System displays password reset form, User enters email, System sends password reset link via Email Service

**Postconditions**: **Success**: User authenticated with active session. User redirected to role-appropriate dashboard. Session token stored. Last login timestamp recorded. Failed login counter reset.<br><br>**Failure**: User not authenticated. Error message displayed. Failed login attempt logged. Account may be temporarily locked if threshold exceeded.

**Outstanding Questions**: 1. Should system support multi-factor authentication (2FA) as optional or mandatory?<br>2. What is the session timeout policy for different user roles?<br>3. Should system support "remember me" functionality for extended sessions?<br>4. Should system log all login attempts or only failed ones?<br>5. Should system notify users via email of successful logins from new devices?<br>6. Should system support single sign-on (SSO) integration for enterprise customers?

---

## UC-03 - Search Hostels

**Source**: `docs/requirements/use-cases/UC-03-search-hostels.md`

**Use Case Name**: Search Hostels

**Use Case ID**: UC-03

**Summary**: Traveler searches for hostels by entering location and dates. System retrieves available hostels matching criteria and displays results with map view. Traveler can filter and sort results to find suitable accommodation.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A3: Traveler. Secondary: A8: Maps API

**Preconditions**: Traveler has internet access. System is operational. At least one hostel property registered and active in system. Maps API service is available.

**Trigger**: Traveler enters location in search box or clicks "Search Hostels" on homepage

**Main Sequence**: 1. Traveler enters search criteria<br>&nbsp;&nbsp;&nbsp;&nbsp;1.1. Traveler enters destination (city name, address, or point of interest)<br>&nbsp;&nbsp;&nbsp;&nbsp;1.2. Traveler selects check-in date<br>&nbsp;&nbsp;&nbsp;&nbsp;1.3. Traveler selects check-out date<br>&nbsp;&nbsp;&nbsp;&nbsp;1.4. Traveler specifies number of guests (optional)<br>2. System sends location query to Maps API<br>3. Maps API returns coordinates and location details<br>4. System retrieves hostels within search area<br>5. System checks availability for specified dates<br>6. System calculates prices for stay duration<br>7. System displays search results<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. System shows list of available hostels<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. System displays map view with hostel markers from Maps API<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. For each hostel, System shows: name, thumbnail photo, price per night, total price, distance from search location, rating, amenities summary<br>8. Traveler applies filters (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Traveler filters by price range<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Traveler filters by amenities (WiFi, parking, breakfast, etc.)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Traveler filters by property type (private room, dorm, entire place)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Traveler filters by guest rating threshold<br>9. System updates search results based on filters<br>10. Traveler sorts results (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Sort by: price (low to high, high to low), distance, rating, popularity<br>11. System reorders results based on selected sort<br>12. Traveler clicks hostel card to view details (continues to UC-04)

**Alternative Sequences**: **Step 3**: If Maps API cannot resolve location, System displays "Location not found" message and suggests similar location names or prompts traveler to refine search<br><br>**Step 4**: If no hostels exist in search area (50km radius), System expands search radius to 100km and displays message "No results in [location]. Showing results within 100km"<br><br>**Step 5**: If no hostels available for specified dates, System displays "No availability for selected dates" and offers option to adjust dates or view fully booked properties<br><br>**Step 9**: If filtered results return zero matches, System displays "No results match your filters" and suggests removing some filters<br><br>**Step 1**: If check-out date is before or equal to check-in date, System displays error "Check-out date must be after check-in date" and prevents search<br><br>**Step 1**: If check-in date is in the past, System displays error "Check-in date cannot be in the past" and resets to today's date<br><br>**Step 3**: If Maps API service unavailable, System performs text-based search using stored location data and displays results without map view

**Postconditions**: **Success**: Search results displayed showing available hostels for specified criteria. Traveler can view details of individual hostels. Search parameters preserved for refinement.<br><br>**Failure**: Error message displayed with guidance. Traveler prompted to adjust search criteria.

**Outstanding Questions**: 1. Should system support "flexible dates" search showing prices across multiple date ranges?<br>2. Should map view display price labels on hostel markers?<br>3. What is the maximum search radius before system shows "no results"?<br>4. Should system save recent searches for quick access?<br>5. Should search results include "sponsored" or "featured" hostels with priority placement?<br>6. Should system support searching by neighborhood or district within a city?<br>7. Should traveler be required to specify dates or allow browsing without dates?

---

## UC-04 - View Hostel Details

**Source**: `docs/requirements/use-cases/UC-04-view-hostel-details.md`

**Use Case Name**: View Hostel Details

**Use Case ID**: UC-04

**Summary**: Traveler views comprehensive details of specific hostel including photos, amenities, room types, pricing, location map, policies, and guest reviews. System retrieves property information and displays in user-friendly format.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A3: Traveler. Secondary: A7: Cloud Storage, A8: Maps API

**Preconditions**: Traveler has accessed system. Hostel exists and is active. Property information available. Cloud Storage and Maps API services operational.

**Trigger**: Traveler clicks hostel card from search results (UC-03) or accesses hostel via direct link

**Main Sequence**: 1. Traveler selects hostel from search results<br>2. System retrieves hostel details<br>3. System requests property photos from Cloud Storage<br>4. System requests location map from Maps API<br>5. System displays hostel details page<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. System shows photo gallery (main image, thumbnails)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. System displays property name, rating, review count<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. System shows location address with interactive map<br>&nbsp;&nbsp;&nbsp;&nbsp;5.4. System lists amenities (WiFi, parking, breakfast, kitchen, lockers, etc.)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.5. System displays property description<br>&nbsp;&nbsp;&nbsp;&nbsp;5.6. System shows available room types with prices<br>&nbsp;&nbsp;&nbsp;&nbsp;5.7. System displays policies (check-in/check-out times, cancellation policy, house rules)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.8. System shows guest reviews with ratings<br>6. Traveler browses photo gallery<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Traveler clicks thumbnail to view full-size photo<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Traveler navigates through photos using arrows<br>7. Traveler explores interactive map<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Traveler zooms map to view nearby attractions<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. System displays points of interest markers via Maps API<br>8. Traveler reviews room type options<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. For each room type, System shows: name, capacity, bed configuration, price per night, availability<br>9. Traveler reads guest reviews<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. System displays reviews sorted by most recent<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. For each review: guest name, rating, date, comment, helpful count<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. Traveler filters reviews by rating (5-star, 4-star, etc.)<br>10. Traveler clicks "Book Now" button (continues to UC-05)

**Alternative Sequences**: **Step 2**: If hostel not found or inactive, System displays "Property not available" message and redirects to search page<br><br>**Step 3**: If Cloud Storage service unavailable, System displays placeholder images and continues with available information<br><br>**Step 4**: If Maps API unavailable, System displays static location information (address, coordinates) without interactive map<br><br>**Step 8**: If no rooms available for traveler's selected dates, System displays "No availability for selected dates" and offers date adjustment<br><br>**Step 9**: If no reviews exist, System displays "No reviews yet. Be the first to review!" message<br><br>**Pre-sequence**: If traveler not selected dates in search, System prompts for check-in/check-out dates before displaying room availability and pricing

**Postconditions**: **Success**: Traveler views complete hostel information. Traveler can proceed to booking or return to search. Page view tracked for analytics.<br><br>**Failure**: Error message displayed. Traveler redirected to search or homepage.

**Outstanding Questions**: 1. Should system display real-time availability count (e.g., "3 rooms left") to create urgency?<br>2. Should traveler be able to save/favorite hostels for later viewing?<br>3. Should system show similar/recommended hostels at bottom of page?<br>4. Should reviews include photos uploaded by guests?<br>5. Should system display "recently viewed" hostels for easy comparison?<br>6. What information requires authentication to view (contact details, exact address)?

---

## UC-05 - Book Accommodation

**Source**: `docs/requirements/use-cases/UC-05-book-accommodation.md`

**Use Case Name**: Book Accommodation

**Use Case ID**: UC-05

**Summary**: Traveler selects room type, provides guest information, and completes payment to book accommodation. System validates information, processes payment via Payment Gateway, creates booking, and sends confirmation via Email Service.

**Dependency**: Include: UC-02 (Log In). Extend: UC-24 (Cancel Booking)

**Actors**: Primary: A3: Traveler. Secondary: A4: Payment Gateway, A5: Email Service

**Preconditions**: Traveler authenticated (via UC-02). Hostel property active and available. Selected dates have availability. Payment Gateway and Email Service operational.

**Trigger**: Traveler clicks "Book Now" button on hostel details page (UC-04)

**Main Sequence**: 1. System verifies traveler authentication<br>2. Include: UC-02 (Log In) if not authenticated<br>3. System displays booking form with pre-filled information<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. System shows selected hostel, room type, dates<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. System displays price breakdown (room rate × nights, taxes, fees)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. System shows total amount<br>4. Traveler reviews and confirms booking details<br>5. Traveler enters guest information<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Traveler enters/confirms full name<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Traveler enters/confirms phone number<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. Traveler enters estimated arrival time<br>&nbsp;&nbsp;&nbsp;&nbsp;5.4. Traveler enters special requests (optional)<br>6. Traveler selects payment method<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Options: Credit/Debit Card, VNPay, SePay, Bank Transfer<br>7. System validates guest information format<br>8. System checks room availability (revalidates real-time)<br>9. System creates pending booking<br>10. System redirects traveler to Payment Gateway<br>11. Traveler completes payment at Payment Gateway<br>&nbsp;&nbsp;&nbsp;&nbsp;11.1. Payment Gateway displays payment interface<br>&nbsp;&nbsp;&nbsp;&nbsp;11.2. Traveler enters payment credentials<br>&nbsp;&nbsp;&nbsp;&nbsp;11.3. Payment Gateway processes transaction<br>12. Payment Gateway sends payment confirmation to System<br>13. System confirms booking<br>14. System sends confirmation email to traveler via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;14.1. Email includes: booking reference number, hostel details, check-in/check-out dates, total amount paid, cancellation policy<br>15. System sends booking notification to hostel owner via Email Service<br>16. System displays booking confirmation page<br>&nbsp;&nbsp;&nbsp;&nbsp;16.1. Shows booking reference number<br>&nbsp;&nbsp;&nbsp;&nbsp;16.2. Shows booking details summary<br>&nbsp;&nbsp;&nbsp;&nbsp;16.3. Provides links to view booking and contact hostel

**Alternative Sequences**: **Step 1**: If traveler not authenticated, System redirects to login page and returns to booking after successful login (Include UC-02)<br><br>**Step 7**: If guest information invalid (phone format incorrect), System displays error messages and returns to Step 5<br><br>**Step 8**: If room no longer available (booked by another traveler), System displays "Room no longer available for selected dates" and offers alternative dates or room types<br><br>**Step 12**: If payment fails (insufficient funds, declined card, transaction timeout), Payment Gateway returns error to System, System cancels pending booking, displays error message, and offers option to retry payment or choose different method<br><br>**Step 12**: If payment processing takes longer than 10 minutes, System displays "Payment pending" status and sends email when payment confirmed<br><br>**Step 14**: If email delivery fails, System logs error, displays warning on confirmation page, and allows traveler to resend confirmation email<br><br>**After Step 16**: Within 24 hours of booking, Traveler may cancel (Extend: UC-24 Cancel Booking)

**Postconditions**: **Success**: Booking confirmed and saved. Payment processed. Confirmation emails sent to traveler and hostel owner. Room availability updated. Booking accessible in traveler's account.<br><br>**Failure**: Booking not created. Payment not processed or refunded if captured. Availability not changed. Error message displayed with resolution guidance.

**Outstanding Questions**: 1. Should system support split payments (multiple travelers sharing cost)?<br>2. Should system offer travel insurance during booking flow?<br>3. What happens if hostel owner rejects booking after payment?<br>4. Should system support installment payments for long-term stays?<br>5. Should system send SMS confirmation in addition to email?<br>6. What is the booking modification policy (change dates, room type)?<br>7. Should system support group bookings (multiple rooms in one transaction)?

---

## UC-06 - Manage Bookings

**Source**: `docs/requirements/use-cases/UC-06-manage-bookings.md`

**Use Case Name**: Manage Bookings

**Use Case ID**: UC-06

**Summary**: Traveler views list of all bookings (upcoming, past, cancelled), accesses booking details, downloads confirmation, and contacts hostel. System retrieves and displays booking information organized by status.

**Dependency**: Include: None. Extend: UC-24 (Cancel Booking) at step 7.4, UC-07 (Submit Review) at step 7.5

**Actors**: Primary: A3: Traveler. Secondary: None

**Preconditions**: Traveler authenticated. Traveler has at least one booking in system. System operational.

**Trigger**: Traveler clicks "My Bookings" link in navigation menu or user profile dropdown

**Main Sequence**: 1. Traveler navigates to bookings page<br>2. System retrieves all bookings for traveler<br>3. System displays bookings organized by tabs<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Tab: Upcoming Bookings (future check-in dates)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Tab: Past Bookings (past check-out dates)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Tab: Cancelled Bookings<br>4. For each booking, System shows summary card<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. Hostel name and thumbnail photo<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. Booking reference number<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. Check-in and check-out dates<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. Room type and number of guests<br>&nbsp;&nbsp;&nbsp;&nbsp;4.5. Total amount paid<br>&nbsp;&nbsp;&nbsp;&nbsp;4.6. Booking status (Confirmed, Checked-In, Completed, Cancelled)<br>5. Traveler selects specific booking to view details<br>6. System displays full booking details page<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Complete hostel information (name, address, phone, email)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Check-in/check-out dates and times<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Room type, bed configuration, number of guests<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Guest information provided at booking<br>&nbsp;&nbsp;&nbsp;&nbsp;6.5. Special requests (if any)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.6. Price breakdown and payment information<br>&nbsp;&nbsp;&nbsp;&nbsp;6.7. Cancellation policy<br>&nbsp;&nbsp;&nbsp;&nbsp;6.8. Booking timeline (booked date, payment date, check-in/out dates)<br>7. Traveler performs booking actions<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Option: Download booking confirmation (PDF)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. Option: Get directions to hostel (opens map)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. Option: Contact hostel (phone or email)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.4. Option: Cancel booking (if within cancellation window) → UC-24<br>&nbsp;&nbsp;&nbsp;&nbsp;7.5. Option: Leave review (if past check-out date and not reviewed) → UC-07<br>8. If action selected, System executes corresponding operation<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Download: System generates PDF confirmation<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Directions: System opens map with hostel location<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Contact: System displays contact options (call, email, message)<br>9. Traveler returns to bookings list or navigates elsewhere

**Alternative Sequences**: **Step 2**: If traveler has no bookings, System displays "No bookings yet" message with link to search hostels<br><br>**Step 3**: If no bookings in specific tab (e.g., no cancelled bookings), System displays empty state message for that tab<br><br>**Step 7.1**: If PDF generation fails, System displays error message and offers option to retry or email confirmation<br><br>**Step 7.4**: If cancellation window expired, System hides cancel option and displays "Non-refundable" status<br><br>**Step 7.5**: If traveler already submitted review, System displays "You reviewed this booking" with link to view review<br><br>**Step 7.5**: If check-out date is future, System hides review option with message "Review available after check-out"

**Postconditions**: **Success**: Traveler views booking information. Actions completed as requested (PDF downloaded, contact initiated, etc.). Traveler can access booking details anytime.<br><br>**Failure**: Error message displayed for failed actions. Booking information remains accessible.

**Outstanding Questions**: 1. Should system send reminder notifications before check-in date?<br>2. Should traveler be able to modify booking dates directly (subject to availability)?<br>3. Should system display weather forecast for check-in dates?<br>4. Should traveler be able to add bookings to calendar (Google Calendar, iCal)?<br>5. Should system show nearby attractions or transportation options?<br>6. Should traveler be able to request early check-in or late check-out through system?<br>7. Should system allow travelers to share booking details with travel companions?

---

## UC-07 - Submit Review

**Source**: `docs/requirements/use-cases/UC-07-submit-review.md`

**Use Case Name**: Submit Review

**Use Case ID**: UC-07

**Summary**: Traveler submits review and rating for hostel after completed stay. System validates review is for past booking by authenticated traveler, publishes review, and updates hostel's aggregate rating.

**Dependency**: Include: UC-02 (Log In). Extend: None

**Actors**: Primary: A3: Traveler. Secondary: None

**Preconditions**: Traveler authenticated (via UC-02). Traveler has completed booking (check-out date passed). Traveler has not already reviewed this booking. System operational.

**Trigger**: Traveler clicks "Leave Review" button on past booking in UC-06, or System prompts traveler via email after check-out

**Main Sequence**: 1. System verifies traveler authentication<br>2. Include: UC-02 (Log In) if not authenticated<br>3. System verifies traveler has completed stay at hostel<br>4. System checks traveler has not already reviewed this booking<br>5. System displays review form<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. System shows booking details (hostel name, dates stayed, room type)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. System displays rating criteria (overall, cleanliness, location, staff, facilities, value)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. System shows text area for written review<br>6. Traveler provides ratings<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Traveler rates overall experience (1-5 stars, required)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Traveler rates individual aspects (1-5 stars each, optional)<br>7. Traveler writes review text<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Minimum 10 characters, maximum 2000 characters<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. Character counter displays remaining characters<br>8. Traveler uploads photos (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Maximum 5 photos<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Supported formats: JPG, PNG<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Maximum 5MB per photo<br>9. Traveler submits review<br>10. System validates review content<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Checks minimum character requirement met<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. Checks for prohibited content (profanity, personal info, spam)<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. Validates photo formats and sizes<br>11. System saves review with pending status<br>12. System sends review to moderation queue<br>13. System displays confirmation message<br>&nbsp;&nbsp;&nbsp;&nbsp;13.1. "Thank you! Your review will be published after moderation (usually within 24 hours)"<br>14. After moderation approval, System publishes review<br>15. System recalculates hostel's aggregate ratings<br>&nbsp;&nbsp;&nbsp;&nbsp;15.1. Updates overall rating average<br>&nbsp;&nbsp;&nbsp;&nbsp;15.2. Updates individual aspect averages<br>&nbsp;&nbsp;&nbsp;&nbsp;15.3. Updates review count<br>16. System marks booking as reviewed<br>17. System displays review on hostel details page (UC-04)

**Alternative Sequences**: **Step 1**: If traveler not authenticated, System redirects to login and returns to review form after successful login (Include UC-02)<br><br>**Step 3**: If check-out date is in future, System displays "You can review this property after your stay ends on [date]" and prevents review submission<br><br>**Step 4**: If traveler already reviewed this booking, System displays existing review with option to edit (within 7 days of original submission)<br><br>**Step 10.2**: If review contains prohibited content, System displays specific error messages (e.g., "Review contains inappropriate language", "Please remove email addresses") and returns to Step 7<br><br>**Step 10.3**: If photo validation fails (wrong format, too large), System displays error for specific photos and allows traveler to remove/replace them<br><br>**Step 14**: If moderation rejects review (spam, fake, inappropriate), System notifies traveler via email with rejection reason and allows resubmission<br><br>**Step 8**: If photo upload fails, System displays error and allows traveler to retry or submit review without photos

**Postconditions**: **Success**: Review submitted and queued for moderation. After approval, review published on hostel page. Hostel ratings updated. Booking marked as reviewed. Traveler receives confirmation.<br><br>**Failure**: Review not saved. Validation errors displayed. Traveler can correct and resubmit.

**Outstanding Questions**: 1. Should system allow hostel owners to respond to reviews?<br>2. Should travelers be able to mark reviews as "helpful" or report inappropriate reviews?<br>3. Should system send reminder emails to travelers who haven't reviewed?<br>4. What is the time limit for submitting reviews after check-out (30 days, 90 days, unlimited)?<br>5. Should system support review translations for international travelers?<br>6. Should system display verification badge showing review is from actual guest?<br>7. Should travelers receive incentives (discount, credits) for submitting reviews?

---

## UC-08 - Manage Profile

**Source**: `docs/requirements/use-cases/UC-08-manage-profile.md`

**Use Case Name**: Manage Profile

**Use Case ID**: UC-08

**Summary**: Traveler views and updates personal profile information including name, email, phone, password, communication preferences, and profile photo. System validates changes and updates profile accordingly.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A3: Traveler. Secondary: None

**Preconditions**: Traveler authenticated. System operational. Traveler has active account.

**Trigger**: Traveler clicks "Profile" or "Account Settings" link in navigation menu or user dropdown

**Main Sequence**: 1. Traveler navigates to profile page<br>2. System retrieves traveler's current profile information<br>3. System displays profile sections<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Personal Information section (name, email, phone, date of birth)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Profile Photo section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Password section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. Communication Preferences section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. Account Security section<br>4. Traveler edits personal information<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. Traveler updates full name<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. Traveler updates phone number<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. Traveler updates date of birth<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. Email display only (changes require verification)<br>5. System validates information format as traveler types<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Real-time validation for phone format<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Age validation (must be 18+)<br>6. Traveler saves changes<br>7. System validates all modified fields<br>8. System updates profile information<br>9. System displays success message "Profile updated successfully"<br>10. System refreshes page with updated information

**Alternative Sequences**: **Step 5**: If information format invalid (phone format incorrect, age under 18), System displays inline error messages and prevents saving<br><br>**Step 7**: If validation fails, System displays specific error messages and highlights problematic fields<br><br>**Profile Photo Flow**:<br>- Traveler clicks "Upload Photo"<br>- Traveler selects image file (JPG, PNG, max 5MB)<br>- System validates image format and size<br>- System displays image preview with crop tool<br>- Traveler adjusts crop area<br>- Traveler saves photo<br>- System stores photo and updates profile<br>- If upload fails: System displays error and allows retry<br><br>**Email Change Flow**:<br>- Traveler clicks "Change Email"<br>- System displays email change form<br>- Traveler enters new email address<br>- Traveler enters current password for verification<br>- System validates password<br>- System sends verification link to new email<br>- System displays "Verification email sent to [new email]"<br>- Traveler clicks verification link in email<br>- System updates email address<br>- If verification not completed within 24 hours: link expires<br><br>**Password Change Flow**:<br>- Traveler clicks "Change Password"<br>- System displays password change form<br>- Traveler enters current password<br>- Traveler enters new password<br>- Traveler confirms new password<br>- System validates current password<br>- System validates new password strength (8+ chars, mixed case, numbers, special chars)<br>- System validates passwords match<br>- System updates password<br>- System displays "Password changed successfully"<br>- System sends security notification email<br>- If current password invalid: display error and offer "Forgot Password" link<br><br>**Communication Preferences Flow**:<br>- Traveler toggles email preferences (booking confirmations, promotional emails, review reminders, newsletter)<br>- Traveler toggles SMS preferences (booking confirmations, urgent notifications)<br>- Traveler selects preferred language (English, Vietnamese)<br>- System saves preferences immediately<br>- System displays "Preferences updated"<br><br>**Account Security Flow**:<br>- Traveler views login history (last 10 logins with dates, locations, devices)<br>- Traveler enables/disables two-factor authentication (if available)<br>- Traveler views active sessions<br>- Traveler logs out other sessions<br>- System terminates selected sessions<br>- System displays "Sessions logged out successfully"

**Postconditions**: **Success**: Profile information updated. Changes reflected throughout system. Traveler receives confirmation. Email/password changes trigger security notifications.<br><br>**Failure**: Profile not updated. Validation errors displayed. Traveler can correct and retry.

**Outstanding Questions**: 1. Should system support social profile links (Facebook, Instagram)?<br>2. Should traveler be able to add emergency contact information?<br>3. Should system support multiple phone numbers (home, mobile)?<br>4. Should system show account activity timeline (bookings, reviews, logins)?<br>5. Should traveler be able to download personal data (GDPR compliance)?<br>6. Should system support account deletion with data retention policy?<br>7. Should profile completion percentage be displayed to encourage complete profiles?

---

## UC-09 - Register Hostel

**Source**: `docs/requirements/use-cases/UC-09-register-hostel.md`

**Use Case Name**: Register Hostel

**Use Case ID**: UC-09

**Summary**: Hostel Owner registers new property by providing business information, property details, and supporting documentation. System validates information, creates property listing, and sends verification confirmation via Email Service.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A1: Hostel Owner. Secondary: A5: Email Service

**Preconditions**: Owner has registered user account and is authenticated. Owner does not have existing pending or active property registration. System operational. Email Service available.

**Trigger**: Owner clicks "Add Property" or "Register Hostel" button in owner dashboard

**Main Sequence**: 1. Owner clicks "Add Property" button<br>2. System displays property registration wizard with progress indicator<br>3. **Step 1: Business Information**<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Owner enters business name<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Owner enters business registration number<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Owner enters tax identification number<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. Owner uploads business license document (PDF, JPG, max 10MB)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. Owner enters business contact information (phone, email)<br>4. Owner clicks "Next" to continue<br>5. System validates business information format<br>6. **Step 2: Property Details**<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Owner enters property name<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Owner enters complete address<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Owner enters property description (minimum 100 characters)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Owner selects property type (hostel, guesthouse, budget hotel)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.5. Owner enters total number of rooms/dorms<br>&nbsp;&nbsp;&nbsp;&nbsp;6.6. Owner enters total guest capacity<br>7. Owner clicks "Next" to continue<br>8. System validates property details<br>9. **Step 3: Amenities & Facilities**<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Owner selects available amenities (WiFi, parking, breakfast, kitchen, lockers, laundry, etc.)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Owner selects common areas (lounge, terrace, garden, game room)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. Owner selects accessibility features (wheelchair access, elevator)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.4. Owner enters check-in/check-out times<br>10. Owner clicks "Next" to continue<br>11. **Step 4: Policies & Rules**<br>&nbsp;&nbsp;&nbsp;&nbsp;11.1. Owner enters cancellation policy details<br>&nbsp;&nbsp;&nbsp;&nbsp;11.2. Owner enters house rules (quiet hours, smoking policy, age restrictions)<br>&nbsp;&nbsp;&nbsp;&nbsp;11.3. Owner enters payment terms (deposit required, accepted methods)<br>&nbsp;&nbsp;&nbsp;&nbsp;11.4. Owner enters minimum/maximum stay requirements (if any)<br>12. Owner clicks "Next" to continue<br>13. **Step 5: Bank Account Information**<br>&nbsp;&nbsp;&nbsp;&nbsp;13.1. Owner enters bank name<br>&nbsp;&nbsp;&nbsp;&nbsp;13.2. Owner enters account number<br>&nbsp;&nbsp;&nbsp;&nbsp;13.3. Owner enters account holder name<br>&nbsp;&nbsp;&nbsp;&nbsp;13.4. Owner uploads bank account verification document<br>14. Owner reviews complete registration summary<br>15. Owner accepts terms and conditions for property listing<br>16. Owner submits registration<br>17. System validates all information completeness<br>18. System creates property listing with "Pending Verification" status<br>19. System sends registration confirmation email to owner via Email Service<br>20. System sends registration details to platform admin for verification review<br>21. System displays confirmation message with next steps<br>&nbsp;&nbsp;&nbsp;&nbsp;21.1. "Registration submitted successfully"<br>&nbsp;&nbsp;&nbsp;&nbsp;21.2. "Verification usually takes 2-3 business days"<br>&nbsp;&nbsp;&nbsp;&nbsp;21.3. Link to complete property setup (add photos, room types)

**Alternative Sequences**: **Step 5**: If business information invalid (wrong format, missing required fields), System displays specific error messages and prevents progression to next step<br><br>**Step 8**: If property address cannot be validated, System displays "Unable to verify address" warning but allows continuation with manual review flag<br><br>**Step 17**: If required information missing, System highlights incomplete sections and returns to specific step<br><br>**After Step 19**: Admin verifies registration within 2-3 business days<br>- **If approved**: System updates property status to "Active", sends approval email via Email Service, enables full property management features<br>- **If additional info needed**: System sends request email via Email Service with specific requirements, owner updates information, resubmits for review<br>- **If rejected**: System updates status to "Rejected", sends rejection email via Email Service with reason, owner can appeal or submit new registration<br><br>**Step 3.4/13.4**: If document upload fails (wrong format, too large), System displays error message specifying requirements and allows retry<br><br>**Step 15**: If owner declines terms and conditions, System prevents submission and displays message "You must accept terms to register property"<br><br>**Wizard Navigation**: Owner clicks "Back" button, System saves current step data and returns to previous step, data preserved for all steps<br><br>**Save as Draft**: Owner clicks "Save and Continue Later", System saves progress, owner can resume registration from last completed step

**Postconditions**: **Success**: Property registration submitted and queued for verification. Owner receives confirmation email. Property listed with "Pending Verification" status. Owner can add photos and room types while awaiting approval.<br><br>**Failure**: Registration not submitted. Validation errors displayed. Draft saved for later completion. Owner can correct and resubmit.

**Outstanding Questions**: 1. Should system support co-ownership (multiple owners for one property)?<br>2. Should system verify business registration number automatically via government API?<br>3. What is the approval rate threshold for expedited verification for established owners?<br>4. Should system support property import from existing booking platforms (Booking.com export)?<br>5. Should owner be able to register property without bank details initially (add later)?<br>6. What documentation is required for properties in different countries/regions?<br>7. Should system offer guided onboarding call for first-time property registration?

---

## UC-10 - Manage Property

**Source**: `docs/requirements/use-cases/UC-10-manage-property.md`

**Use Case Name**: Manage Property

**Use Case ID**: UC-10

**Summary**: Hostel Owner updates property information including description, photos, amenities, policies, and location details. System uploads photos to Cloud Storage, geocodes address via Maps API, and updates property listing visible to travelers.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A1: Hostel Owner. Secondary: A7: Cloud Storage, A8: Maps API

**Preconditions**: Owner authenticated. Owner has registered property (UC-09) with "Active" or "Pending Verification" status. Cloud Storage and Maps API services operational.

**Trigger**: Owner clicks "Manage Property" or "Edit Property" button in property dashboard

**Main Sequence**: 1. Owner navigates to property management page<br>2. System retrieves current property information<br>3. System displays property management sections<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Basic Information section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Photos & Media section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Description & Highlights section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. Amenities & Facilities section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. Location & Directions section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.6. Policies & Rules section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.7. Operating Hours section<br>4. **Basic Information Management**<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. Owner updates property name<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. Owner updates address<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. Owner updates contact phone and email<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. Owner updates property type/category<br>&nbsp;&nbsp;&nbsp;&nbsp;4.5. System validates format as owner types<br>&nbsp;&nbsp;&nbsp;&nbsp;4.6. If address changed: System sends address to Maps API for geocoding<br>&nbsp;&nbsp;&nbsp;&nbsp;4.7. Maps API returns coordinates<br>&nbsp;&nbsp;&nbsp;&nbsp;4.8. System updates property location<br>5. **Photos & Media Management**<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. System displays current photo gallery with thumbnails<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Owner uploads new photos<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. System validates photos (format: JPG/PNG, size: max 10MB each, min resolution: 1200x800)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.4. System uploads photos to Cloud Storage<br>&nbsp;&nbsp;&nbsp;&nbsp;5.5. Cloud Storage returns photo URLs<br>&nbsp;&nbsp;&nbsp;&nbsp;5.6. System adds photos to gallery<br>&nbsp;&nbsp;&nbsp;&nbsp;5.7. Owner reorders photos by drag-and-drop<br>&nbsp;&nbsp;&nbsp;&nbsp;5.8. Owner sets primary/cover photo<br>&nbsp;&nbsp;&nbsp;&nbsp;5.9. Owner deletes unwanted photos<br>&nbsp;&nbsp;&nbsp;&nbsp;5.10. System removes deleted photos from Cloud Storage<br>&nbsp;&nbsp;&nbsp;&nbsp;5.11. Owner adds photo captions/descriptions (optional)<br>6. **Description & Highlights**<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Owner updates main property description<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Character counter displays remaining characters (max 2000)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Owner adds/edits property highlights (unique features, nearby attractions)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Owner updates "About the Neighborhood" section<br>7. **Amenities & Facilities**<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. System displays checklist of amenity categories<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. Owner checks/unchecks amenities (WiFi, parking, breakfast, kitchen, etc.)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. Owner specifies amenity details (free WiFi, paid parking, buffet breakfast)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.4. Owner updates common area descriptions<br>8. **Location & Directions**<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. System displays interactive map from Maps API showing property location<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Owner adjusts map marker position if needed<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Owner adds directions from airport/train station/bus terminal<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Owner lists nearby points of interest (manually entered)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.5. System requests nearby places from Maps API<br>&nbsp;&nbsp;&nbsp;&nbsp;8.6. Maps API returns nearby attractions, restaurants, transit<br>&nbsp;&nbsp;&nbsp;&nbsp;8.7. System displays nearby places on map<br>9. **Policies & Rules**<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Owner updates cancellation policy details<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Owner updates house rules<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. Owner updates check-in/check-out times and instructions<br>&nbsp;&nbsp;&nbsp;&nbsp;9.4. Owner updates payment and deposit policies<br>10. Owner saves changes<br>11. System validates all modified information<br>12. System updates property listing<br>13. System displays success message "Property updated successfully"<br>14. System publishes changes immediately (if property active) or saves as draft (if pending verification)

**Alternative Sequences**: **Step 4.6-4.8**: If Maps API cannot geocode address, System displays "Unable to locate address on map" warning, allows manual marker placement, flags for admin review<br><br>**Step 5.3**: If photo validation fails (wrong format, too large, low resolution), System displays specific error for each photo and allows owner to retry or skip<br><br>**Step 5.4**: If Cloud Storage upload fails, System retries upload up to 3 times, displays error if all retries fail, allows owner to retry later<br><br>**Step 11**: If validation fails (required fields empty, invalid data), System displays error messages highlighting problems and prevents saving<br><br>**Property Status Toggle**:<br>- Owner clicks "Pause Listing" to temporarily hide property from search results<br>- System updates property status to "Paused"<br>- System displays "Property hidden from travelers. Bookings paused."<br>- Existing confirmed bookings remain valid<br>- Owner clicks "Activate Listing" to restore visibility<br><br>**Bulk Photo Upload**:<br>- Owner selects multiple photos (up to 20) simultaneously<br>- System displays upload progress for each photo<br>- System shows success/failure status for each upload<br>- Failed uploads can be retried individually<br><br>**Photo Optimization**:<br>- System detects photo orientation issues<br>- System displays "Rotate photo?" prompt<br>- Owner rotates photo before upload<br>- System uploads corrected version to Cloud Storage<br><br>**Unsaved Changes Warning**:<br>- Owner navigates away without saving<br>- System displays "You have unsaved changes. Discard changes?"<br>- Owner confirms or cancels navigation

**Postconditions**: **Success**: Property information updated and saved. Photos stored in Cloud Storage. Location geocoded via Maps API. Changes visible to travelers immediately (if active). Owner receives confirmation.<br><br>**Failure**: Changes not saved. Validation errors displayed. Photos not uploaded. Owner can correct and retry.

**Outstanding Questions**: 1. Should system support video uploads for property tours?<br>2. Should system automatically suggest nearby attractions via Maps API?<br>3. Should owner be able to schedule property information updates (e.g., seasonal descriptions)?<br>4. Should system provide photo quality analysis and improvement suggestions?<br>5. Should system support 360-degree photos or virtual tours?<br>6. Should property changes require admin approval if major (address change, name change)?<br>7. Should system automatically translate descriptions to multiple languages?

---

## UC-11 - Manage Room Types

**Source**: `docs/requirements/use-cases/UC-11-manage-room-types.md`

**Use Case Name**: Manage Room Types

**Use Case ID**: UC-11

**Summary**: Hostel Owner creates and manages room type inventory including private rooms and dorm beds. Owner configures capacity, pricing, bed configurations, availability calendar, and room-specific amenities. System calculates occupancy and manages booking availability.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A1: Hostel Owner. Secondary: None

**Preconditions**: Owner authenticated. Owner has registered property (UC-09). Property status is "Active" or "Pending Verification". System operational.

**Trigger**: Owner clicks "Manage Rooms" or "Room Inventory" button in property dashboard

**Main Sequence**: 1. Owner navigates to room management page<br>2. System retrieves existing room types for property<br>3. System displays room types list<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. For each room: name, type, capacity, pricing, availability status<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Quick action buttons: Edit, Duplicate, Disable, Delete<br>4. Owner clicks "Add Room Type" button<br>5. System displays room type creation form<br>6. **Basic Room Information**<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Owner enters room type name (e.g., "4-Bed Mixed Dorm", "Private Double Room")<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Owner selects room category (Private Room, Shared Dorm, Entire Place)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Owner enters room size in square meters (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Owner enters number of rooms of this type (quantity)<br>7. **Bed Configuration**<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. If Private Room: Owner specifies bed type (single, double, queen, king, twin)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. If Shared Dorm: Owner specifies total beds per room<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. Owner specifies bed types (bunk bed, single bed, capsule)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.4. System calculates total guest capacity<br>8. **Pricing Configuration**<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Owner enters base price per night<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. If Private Room: price is per room<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. If Shared Dorm: price is per bed per night<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Owner configures seasonal pricing (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.5. Owner enters weekend surcharge percentage (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.6. Owner sets minimum/maximum stay requirements<br>9. **Room-Specific Amenities**<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Owner selects amenities (private bathroom, air conditioning, balcony, window, locker, reading light)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Owner adds room description (highlights, views, special features)<br>10. **Availability Settings**<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Owner sets default availability (available/blocked)<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. Owner blocks specific dates on calendar (maintenance, reserved for groups)<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. System displays availability calendar with color coding<br>11. **Guest Restrictions (Optional)**<br>&nbsp;&nbsp;&nbsp;&nbsp;11.1. Owner specifies age restrictions (18+, families only)<br>&nbsp;&nbsp;&nbsp;&nbsp;11.2. Owner specifies gender restrictions (female only, male only, mixed)<br>&nbsp;&nbsp;&nbsp;&nbsp;11.3. Owner sets maximum occupancy per booking<br>12. Owner saves room type<br>13. System validates room configuration<br>14. System creates room type record<br>15. System generates individual bed/room inventory<br>&nbsp;&nbsp;&nbsp;&nbsp;15.1. For Private Rooms: Creates X room units (where X = quantity)<br>&nbsp;&nbsp;&nbsp;&nbsp;15.2. For Dorms: Creates X rooms, each with Y beds (where Y = beds per room)<br>16. System displays success message "Room type created successfully"<br>17. System returns to room types list showing new addition

**Alternative Sequences**: **Step 13**: If validation fails (missing required fields, pricing below minimum threshold), System displays specific error messages and prevents saving<br><br>**Edit Existing Room Type**:<br>- Owner clicks "Edit" on existing room<br>- System displays edit form with current data<br>- Owner modifies information<br>- System validates changes<br>- If active bookings exist: System prevents changes that conflict with bookings (reducing capacity, changing pricing retroactively)<br>- System saves changes<br>- Changes apply to future bookings only<br><br>**Duplicate Room Type**:<br>- Owner clicks "Duplicate" on existing room<br>- System creates copy with "(Copy)" appended to name<br>- Owner modifies as needed<br>- Owner saves new room type<br>- Useful for creating similar rooms with minor differences<br><br>**Disable Room Type**:<br>- Owner clicks "Disable" to temporarily remove from availability<br>- System updates status to "Disabled"<br>- System hides room from traveler search results<br>- Existing bookings remain valid<br>- Owner can re-enable anytime<br><br>**Delete Room Type**:<br>- Owner clicks "Delete"<br>- System checks for active or upcoming bookings<br>- If bookings exist: System prevents deletion, displays "Cannot delete room type with active bookings"<br>- If no bookings: System displays confirmation "Are you sure? This cannot be undone."<br>- Owner confirms deletion<br>- System marks room type as deleted (soft delete)<br><br>**Bulk Availability Update**:<br>- Owner selects date range on calendar<br>- Owner marks as "Available" or "Blocked"<br>- System updates all selected dates<br>- Useful for seasonal closures or maintenance periods<br><br>**Pricing Rules**:<br>- Owner creates seasonal pricing rule (high season, holidays)<br>- Owner specifies date range<br>- Owner enters adjusted price or percentage increase<br>- System applies pricing rule automatically for bookings in that period<br>- System displays effective price in calendar view<br><br>**Room Photos**:<br>- Owner uploads room-specific photos (separate from property photos)<br>- Photos displayed when traveler views room type details<br>- Owner reorders photos for best presentation

**Postconditions**: **Success**: Room type created/updated and available for booking. Inventory generated for individual rooms/beds. Pricing configured. Availability calendar set. Changes visible to travelers in search results.<br><br>**Failure**: Room type not saved. Validation errors displayed. Inventory unchanged. Owner can correct and retry.

**Outstanding Questions**: 1. Should system support dynamic pricing based on demand/occupancy?<br>2. Should owner be able to set different prices for weekdays vs weekends automatically?<br>3. Should system recommend pricing based on competitor analysis?<br>4. Should system support package deals (e.g., 7 nights for price of 6)?<br>5. Should owner be able to allocate specific room numbers/identifiers?<br>6. Should system support last-minute discounts automatically?<br>7. Should owner be able to link rooms (e.g., connecting rooms for families)?

---

## UC-12 - Process Booking

**Source**: `docs/requirements/use-cases/UC-12-process-booking.md`

**Use Case Name**: Process Booking

**Use Case ID**: UC-12

**Summary**: Hostel Owner views incoming bookings, confirms or declines reservations, assigns beds, updates booking status, and communicates with guests. System sends notifications via Email Service and SMS Gateway at key booking milestones.

**Dependency**: Include: None. Extend: None. Related to: UC-05 (Book Accommodation)

**Actors**: Primary: A1: Hostel Owner. Secondary: A5: Email Service, A6: SMS Gateway

**Preconditions**: Owner authenticated. Owner has active hostel property. Booking exists from traveler (UC-05). Email Service and SMS Gateway operational.

**Trigger**: Owner receives booking notification via email/SMS, or Owner checks booking dashboard

**Main Sequence**: 1. Owner navigates to bookings dashboard<br>2. System retrieves all bookings for owner's property<br>3. System displays bookings organized by status<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Tab: New Bookings (awaiting confirmation)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Tab: Confirmed Bookings (upcoming)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Tab: Checked-In (currently staying)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. Tab: Completed (past check-out)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. Tab: Cancelled<br>4. For each booking, System displays summary card<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. Guest name and contact<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. Booking reference number<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. Check-in and check-out dates<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. Room type and number of guests<br>&nbsp;&nbsp;&nbsp;&nbsp;4.5. Total amount<br>&nbsp;&nbsp;&nbsp;&nbsp;4.6. Booking status badge<br>&nbsp;&nbsp;&nbsp;&nbsp;4.7. Time until check-in<br>5. Owner selects specific booking to view details<br>6. System displays comprehensive booking details<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Guest information (name, email, phone, nationality)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Booking timeline (booked date, confirmed date, check-in/out dates)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Room details (type, bed configuration, number of guests)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Special requests from guest<br>&nbsp;&nbsp;&nbsp;&nbsp;6.5. Payment information (amount, payment method, transaction ID)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.6. Bed assignment section (if applicable for dorm rooms)<br>7. Owner confirms new booking<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Owner clicks "Confirm Booking" button<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. System validates room still available<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. System updates booking status to "Confirmed"<br>&nbsp;&nbsp;&nbsp;&nbsp;7.4. System sends confirmation email to guest via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;7.5. System sends SMS confirmation to guest via SMS Gateway<br>&nbsp;&nbsp;&nbsp;&nbsp;7.6. System displays "Booking confirmed successfully"<br>8. For dorm bookings, Owner assigns specific beds<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. System displays bed assignment interface with room layout<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. System shows available beds (green), occupied beds (red), selected beds (blue)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Owner selects bed(s) for guest<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. System validates bed availability<br>&nbsp;&nbsp;&nbsp;&nbsp;8.5. System saves bed assignment<br>&nbsp;&nbsp;&nbsp;&nbsp;8.6. System updates booking with bed numbers<br>9. Owner adds internal notes to booking (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Owner enters notes (early check-in request, VIP guest, special dietary needs)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. System saves notes (visible only to owner/staff)<br>10. Owner communicates with guest via system messaging<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Owner clicks "Message Guest" button<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. System displays message compose form<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. Owner types message<br>&nbsp;&nbsp;&nbsp;&nbsp;10.4. System sends message to guest via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;10.5. System saves message in booking communication history

**Alternative Sequences**: **Step 7**: If Owner declines booking (overbooking, property issue), Owner clicks "Decline Booking", enters reason, System cancels booking, processes refund via Payment Gateway, sends cancellation notification to guest with reason and refund details<br><br>**Step 7.2**: If room no longer available (double-booked by mistake), System displays error "Room no longer available", prevents confirmation, Owner must decline booking with refund<br><br>**Step 8.4**: If selected bed already assigned to another guest (race condition), System displays "Bed no longer available", highlights available beds, Owner selects different bed<br><br>**No-Show Flow**:<br>- Check-in time passes without guest arrival<br>- Owner marks booking as "No-Show"<br>- System updates booking status<br>- System sends no-show notification to guest via Email Service<br>- System applies no-show policy (no refund as per policy)<br>- Bed/room released for new bookings<br><br>**Early Check-In Request**:<br>- Guest contacts owner requesting early check-in<br>- Owner checks room availability<br>- If available: Owner approves, updates check-in time, sends confirmation<br>- If not available: Owner declines with explanation, offers luggage storage<br><br>**Modification Request**:<br>- Guest requests booking changes (extend stay, change room type)<br>- Owner checks availability for requested changes<br>- If available: Owner quotes price difference, guest confirms, Owner updates booking<br>- If not available: Owner declines with alternatives<br><br>**Urgent Notification Flow**:<br>- For bookings within 24 hours: System sends SMS reminder to Owner via SMS Gateway<br>- For new bookings: System sends immediate SMS alert via SMS Gateway<br>- For cancellations: System sends urgent SMS notification via SMS Gateway

**Postconditions**: **Success**: Booking status updated. Guest notified of changes. Bed assignments saved. Communication logged. Owner dashboard reflects current status.<br><br>**Failure**: Booking status unchanged. Error displayed. Guest not notified until issue resolved.

**Outstanding Questions**: 1. Should system auto-confirm bookings after 24 hours if owner doesn't respond?<br>2. Should system support bulk operations (confirm multiple bookings at once)?<br>3. Should owner be able to set auto-confirmation rules (e.g., always confirm bookings >7 days out)?<br>4. Should system display revenue projections based on confirmed bookings?<br>5. Should system send reminder to owner for bookings awaiting confirmation?<br>6. What happens if owner's property becomes unavailable (maintenance, emergency)?<br>7. Should system support staff permissions (receptionist can confirm but not decline)?

---

## UC-13 - Manage Staff

**Source**: `docs/requirements/use-cases/UC-13-manage-staff.md`

**Use Case Name**: Manage Staff

**Use Case ID**: UC-13

**Summary**: Hostel Owner invites staff members, assigns roles and permissions, manages active staff accounts, and removes staff access. System sends invitation emails via Email Service and enforces role-based access control.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A1: Hostel Owner. Secondary: A5: Email Service

**Preconditions**: Owner authenticated. Owner has active property. System operational. Email Service available.

**Trigger**: Owner clicks "Manage Staff" or "Team" button in property dashboard

**Main Sequence**: 1. Owner navigates to staff management page<br>2. System retrieves current staff list for property<br>3. System displays staff members table<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. For each staff: name, email, role, status (active/pending), last login, actions<br>4. Owner clicks "Invite Staff Member" button<br>5. System displays staff invitation form<br>6. Owner enters staff information<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Owner enters staff email address<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Owner enters staff full name<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Owner selects role (Receptionist, Manager, Owner)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Owner configures permissions<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.1. View bookings (read-only)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.2. Manage bookings (confirm, decline, modify)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.3. Check-in/check-out guests<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.4. View analytics reports<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.5. Manage property settings<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6.4.6. Manage room types and pricing<br>7. System validates email format<br>8. System checks if email already associated with property staff<br>9. System creates pending staff invitation<br>10. System sends invitation email to staff via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Email includes: property name, inviting owner name, role assigned, registration link, expiration (7 days)<br>11. System displays confirmation "Invitation sent to [email]"<br>12. Staff member receives invitation email<br>13. Staff member clicks registration link<br>14. System validates invitation token<br>15. System displays staff registration form<br>16. Staff member creates account or logs in with existing account<br>17. System associates staff account with property<br>18. System applies assigned role and permissions<br>19. System sends confirmation email to staff and owner via Email Service<br>20. System updates staff status to "Active"<br>21. Staff member can now access property dashboard with assigned permissions

**Alternative Sequences**: **Step 8**: If email already exists as staff for this property, System displays error "This email is already associated with your property" and prevents duplicate invitation<br><br>**Step 14**: If invitation expired (>7 days), System displays "Invitation expired" and offers option to request new invitation from owner<br><br>**Step 14**: If invitation already used, System displays "Invitation already accepted" with link to login<br><br>**Edit Staff Permissions**:<br>- Owner clicks "Edit" on active staff member<br>- System displays permission editing form<br>- Owner modifies role or specific permissions<br>- System validates permission changes<br>- System saves updated permissions<br>- Changes apply immediately<br>- System sends notification email to staff about permission changes<br><br>**Suspend Staff Access**:<br>- Owner clicks "Suspend" on active staff<br>- System displays confirmation dialog<br>- Owner confirms suspension<br>- System updates staff status to "Suspended"<br>- System revokes access tokens<br>- Staff cannot log in to property dashboard<br>- Owner can reactivate anytime<br>- System sends notification email to staff<br><br>**Remove Staff Member**:<br>- Owner clicks "Remove" on staff<br>- System displays confirmation "Remove [name] from your property? They will lose all access."<br>- Owner confirms removal<br>- System checks for pending tasks assigned to staff<br>- System removes staff association with property<br>- System sends notification email to removed staff<br>- Staff account remains but loses property access<br><br>**Resend Invitation**:<br>- Owner clicks "Resend" on pending invitation<br>- System generates new invitation token<br>- System sends new invitation email via Email Service<br>- Previous invitation token invalidated<br><br>**Cancel Invitation**:<br>- Owner clicks "Cancel" on pending invitation<br>- System invalidates invitation token<br>- System removes pending invitation<br>- No notification sent to invitee<br><br>**Staff Self-Registration Disabled**:<br>- If staff email not invited by owner<br>- Staff cannot add themselves to property<br>- All staff must be explicitly invited by property owner

**Postconditions**: **Success**: Staff invitation sent or staff permissions updated. Staff member receives access with configured permissions. Owner retains full control. Activity logged.<br><br>**Failure**: Invitation not sent. Permission changes not applied. Error messages displayed. Owner can retry.

**Outstanding Questions**: 1. Should system support staff schedules/shifts management?<br>2. Should owner be able to set permission expiration dates (temporary access)?<br>3. Should staff receive daily/weekly activity summaries?<br>4. Should system support staff departments (housekeeping, front desk)?<br>5. Should staff be able to request permission changes from owner?<br>6. Should system track staff performance metrics (bookings processed, response times)?<br>7. Should owner be able to delegate staff management to a manager role?

---

## UC-14 - View Analytics

**Source**: `docs/requirements/use-cases/UC-14-view-analytics.md`

**Use Case Name**: View Analytics

**Use Case ID**: UC-14

**Summary**: Hostel Owner views property performance analytics including revenue, occupancy, booking trends, guest demographics, and review ratings. System aggregates data and displays visualizations for business insights.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A1: Hostel Owner. Secondary: None

**Preconditions**: Owner authenticated. Owner has active property with booking history. System operational.

**Trigger**: Owner clicks "Analytics" or "Reports" button in property dashboard

**Main Sequence**: 1. Owner navigates to analytics page<br>2. System retrieves property data for selected time period (default: last 30 days)<br>3. System displays analytics dashboard with sections<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. **Overview KPIs** (key metrics at top)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. **Revenue Analytics** section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. **Occupancy Analytics** section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. **Booking Analytics** section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. **Guest Analytics** section<br>&nbsp;&nbsp;&nbsp;&nbsp;3.6. **Review Analytics** section<br>4. **Overview KPIs Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. Total Revenue (with % change vs previous period)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. Average Daily Rate (ADR)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. Occupancy Rate (%)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. Total Bookings (with % change)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.5. Average Rating (with trend indicator)<br>&nbsp;&nbsp;&nbsp;&nbsp;4.6. Cancellation Rate (%)<br>5. **Revenue Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Revenue trend chart (daily/weekly/monthly)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Revenue by room type (pie chart)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. Revenue forecast (next 30 days based on confirmed bookings)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.4. Commission breakdown (platform fees deducted)<br>6. **Occupancy Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Occupancy rate trend chart<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Occupancy by room type (bar chart)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Peak vs off-peak occupancy comparison<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Occupancy calendar heat map (color-coded by % occupied)<br>7. **Booking Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Booking source breakdown (direct, search, referral)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. Booking lead time distribution (how far in advance guests book)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. Average length of stay<br>&nbsp;&nbsp;&nbsp;&nbsp;7.4. Cancellation rate and reasons (if provided)<br>&nbsp;&nbsp;&nbsp;&nbsp;7.5. Booking conversion rate (views → bookings)<br>8. **Guest Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Guest nationality breakdown (top 10 countries)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Guest age distribution (18-25, 26-35, 36-45, 46+)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Repeat guest rate (%)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Group size distribution (solo, couple, group)<br>9. **Review Analytics Display**<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Average rating trend chart<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Rating breakdown by category (cleanliness, location, staff, value)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. Review sentiment analysis (positive, neutral, negative word cloud)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.4. Response rate to reviews (%)<br>10. Owner adjusts time period filter<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Options: Last 7 days, 30 days, 90 days, Year to date, Custom range<br>11. System recalculates analytics for selected period<br>12. System updates all charts and metrics<br>13. Owner exports report (optional)<br>&nbsp;&nbsp;&nbsp;&nbsp;13.1. Owner clicks "Export Report" button<br>&nbsp;&nbsp;&nbsp;&nbsp;13.2. Owner selects format (PDF, Excel)<br>&nbsp;&nbsp;&nbsp;&nbsp;13.3. System generates report file<br>&nbsp;&nbsp;&nbsp;&nbsp;13.4. System downloads report to owner's device

**Alternative Sequences**: **Step 2**: If property has no booking history, System displays "No data available yet" with tips to get first bookings and sample dashboard preview<br><br>**Step 10**: If custom date range selected, Owner specifies start and end dates via calendar picker, System validates date range (max 365 days), calculates analytics<br><br>**Step 13**: If export fails (report too large, timeout), System displays error and suggests narrower date range or simplified report<br><br>**Comparison Mode**:<br>- Owner enables "Compare with previous period" toggle<br>- System displays side-by-side comparison<br>- All metrics show % change (increase/decrease)<br>- Charts display two lines (current vs previous period)<br>- Useful for year-over-year or month-over-month analysis<br><br>**Drill-Down View**:<br>- Owner clicks specific data point on chart<br>- System displays detailed breakdown for that segment<br>- Example: Click on "Revenue by Room Type" pie slice → shows daily revenue for that room type<br>- Owner returns to overview via breadcrumb navigation<br><br>**Competitor Benchmarking** (future enhancement):<br>- System displays anonymous aggregated data from similar properties<br>- Shows how owner's metrics compare to market average<br>- Helps owner identify competitive advantages or improvement areas<br><br>**Goal Setting**:<br>- Owner sets revenue/occupancy goals for upcoming periods<br>- System tracks progress toward goals<br>- Dashboard shows goal progress bars<br>- Alerts when goals achieved or at risk<br><br>**Scheduled Reports**:<br>- Owner configures automatic report delivery<br>- Selects frequency (daily, weekly, monthly)<br>- System sends report via email at scheduled time<br>- Useful for passive monitoring without logging in

**Postconditions**: **Success**: Owner views comprehensive analytics insights. Data visualized clearly. Reports exported if requested. Informed business decisions enabled.<br><br>**Failure**: Error message displayed if data retrieval fails. Partial data shown with notice. Owner can retry or contact support.

**Outstanding Questions**: 1. Should system provide AI-powered insights and recommendations?<br>2. Should owner be able to create custom reports with selected metrics?<br>3. Should system send automated alerts for anomalies (sudden occupancy drop)?<br>4. Should analytics include financial forecasting models?<br>5. Should system track marketing campaign effectiveness (if supported)?<br>6. Should owner be able to share analytics with investors or partners?<br>7. Should system provide API access for custom analytics integrations?

---

## UC-15 - Manage Subscription

**Source**: `docs/requirements/use-cases/UC-15-manage-subscription.md`

**Use Case Name**: Manage Subscription

**Use Case ID**: UC-15

**Summary**: Hostel Owner views current subscription plan, upgrades or downgrades plan, updates payment method, and reviews billing history. System processes payments via Payment Gateway and adjusts feature access based on active plan.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A1: Hostel Owner. Secondary: A4: Payment Gateway

**Preconditions**: Owner authenticated. Owner has active property. Payment Gateway operational. System operational.

**Trigger**: Owner clicks "Subscription" or "Billing" button in account settings, or System prompts owner when trial period ending

**Main Sequence**: 1. Owner navigates to subscription management page<br>2. System retrieves current subscription details<br>3. System displays subscription dashboard<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Current plan name and pricing<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Billing cycle (monthly/annual)<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. Next billing date<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. Plan features included<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. Current usage vs plan limits<br>4. System displays available subscription plans<br>&nbsp;&nbsp;&nbsp;&nbsp;4.1. **Free Plan**: Single property, basic features, 5% commission<br>&nbsp;&nbsp;&nbsp;&nbsp;4.2. **Starter Plan** (₫299,000/month): Enhanced features, 3.5% commission, email support<br>&nbsp;&nbsp;&nbsp;&nbsp;4.3. **Professional Plan** (₫599,000/month): Advanced features, 2.5% commission, priority support, analytics<br>&nbsp;&nbsp;&nbsp;&nbsp;4.4. **Enterprise Plan** (₫1,499,000/month): Premium features, 1.5% commission, dedicated account manager, API access<br>5. Owner selects plan to view details<br>6. System displays plan comparison table<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Features by plan (checkmarks for included features)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Commission rates<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Support level<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Property limits<br>&nbsp;&nbsp;&nbsp;&nbsp;6.5. Staff account limits<br>7. Owner clicks "Upgrade" or "Downgrade" button<br>8. System displays plan change confirmation<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. New plan details<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Pro-rated amount (if mid-cycle upgrade)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Next billing amount and date<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Feature changes summary<br>9. Owner confirms plan change<br>10. System processes payment via Payment Gateway<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. If upgrade with immediate charge: Processes pro-rated payment<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. If downgrade: Schedules change for next billing cycle<br>11. Payment Gateway processes transaction<br>12. Payment Gateway sends confirmation to System<br>13. System updates subscription plan<br>14. System adjusts feature access immediately<br>15. System sends confirmation email with receipt<br>16. System displays success message "Subscription updated to [Plan Name]"

**Alternative Sequences**: **Step 10**: If payment fails (insufficient funds, expired card), Payment Gateway returns error, System displays payment failure message, Owner can update payment method and retry<br><br>**Free Trial Flow**:<br>- New property owners start with 14-day Professional Plan trial<br>- Trial period countdown displayed in dashboard<br>- 7 days before expiration: System sends reminder email<br>- 1 day before expiration: System sends urgent reminder<br>- On expiration: System downgrades to Free Plan if no payment method added<br>- Owner retains data but loses premium features<br><br>**Update Payment Method**:<br>- Owner clicks "Update Payment Method"<br>- System displays payment method form<br>- Owner enters new credit card details<br>- System sends to Payment Gateway for validation<br>- Payment Gateway tokenizes card (PCI compliant)<br>- System saves payment method token<br>- System displays "Payment method updated successfully"<br><br>**Cancel Subscription**:<br>- Owner clicks "Cancel Subscription"<br>- System displays cancellation confirmation with consequences<br>- Owner selects cancellation reason from dropdown<br>- Owner confirms cancellation<br>- System schedules cancellation for end of current billing period<br>- System sends cancellation confirmation email<br>- Owner retains access until period end<br>- At period end: System downgrades to Free Plan<br>- All premium features disabled<br><br>**View Billing History**:<br>- Owner clicks "Billing History" tab<br>- System displays invoice list (date, amount, plan, status, receipt link)<br>- Owner downloads invoice PDF<br>- System generates invoice with VAT details<br><br>**Annual Billing Discount**:<br>- Owner toggles "Billed Annually" switch<br>- System displays annual pricing (15% discount vs monthly)<br>- Owner confirms annual billing<br>- System processes upfront annual payment<br>- Next billing date set to 12 months ahead<br><br>**Reactivate Cancelled Subscription**:<br>- Owner previously cancelled subscription<br>- Owner clicks "Reactivate" on preferred plan<br>- System processes immediate payment<br>- Features restored immediately<br>- Billing cycle restarts<br><br>**Feature Usage Limits Exceeded**:<br>- Owner reaches plan limit (e.g., maximum properties)<br>- System prevents action with upgrade prompt<br>- "Upgrade to Starter plan to add more properties"<br>- Owner can upgrade immediately from prompt

**Postconditions**: **Success**: Subscription plan updated. Payment processed. Features adjusted. Owner receives confirmation and receipt. Billing cycle updated.<br><br>**Failure**: Plan not changed. Payment not processed. Error message displayed. Owner can retry or contact support.

**Outstanding Questions**: 1. Should system offer multi-property discount packages?<br>2. Should system support add-ons (extra staff accounts, premium support) separately from plan tiers?<br>3. Should owner be able to prepay for multiple months with discount?<br>4. Should system offer refunds for cancellations (pro-rated, full, none)?<br>5. Should system support payment methods beyond credit cards (bank transfer, PayPal)?<br>6. Should system offer loyalty discounts for long-term customers?<br>7. Should system support referral credits (discount for referring other owners)?

---

## UC-16 - Communicate With Guest

**Source**: `docs/requirements/use-cases/UC-16-communicate-with-guest.md`

**Use Case Name**: Communicate With Guest

**Use Case ID**: UC-16

**Summary**: Hostel Owner sends messages to guests regarding bookings, provides check-in instructions, answers questions, and maintains communication history. System delivers messages via Email Service and stores conversation threads.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A1: Hostel Owner. Secondary: A5: Email Service

**Preconditions**: Owner authenticated. Booking exists with guest information. Guest has valid email address. Email Service operational.

**Trigger**: Owner clicks "Message Guest" button from booking details page (UC-12), or Guest sends inquiry about property

**Main Sequence**: 1. Owner navigates to messages inbox<br>2. System retrieves message threads for property<br>3. System displays inbox with conversation list<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. For each thread: guest name, booking reference, last message preview, unread badge, timestamp<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. Filters: All, Unread, Archived<br>4. Owner selects conversation thread or starts new message<br>5. System displays message thread view<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Full conversation history (messages from owner and guest)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Associated booking details sidebar (dates, room type, amount)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. Guest profile snippet (name, nationality, past bookings count)<br>6. Owner composes new message<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Owner types message in text area<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Owner can use quick reply templates (check-in instructions, directions, policies)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Owner attaches files if needed (PDF directions, house rules) (optional)<br>7. Owner sends message<br>8. System validates message content<br>9. System saves message to thread<br>10. System sends email to guest via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Email includes message content<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. Email includes "Reply to this email" option<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. Email includes link to view full thread in traveler account<br>11. System marks thread as "Owner responded"<br>12. System displays sent confirmation<br>13. Guest receives email notification<br>14. Guest replies via email or through traveler account<br>15. System receives guest reply<br>16. System adds reply to conversation thread<br>17. System marks thread as "Unread" for owner<br>18. System sends real-time notification to owner (if online)<br>19. Owner views guest reply and continues conversation

**Alternative Sequences**: **Step 8**: If message empty or contains only whitespace, System displays error "Message cannot be empty" and prevents sending<br><br>**Quick Reply Templates**:<br>- Owner clicks "Quick Reply" button<br>- System displays template categories (Check-in, Directions, Policies, FAQ)<br>- Owner selects template<br>- System inserts template text into message field<br>- Owner customizes text as needed<br>- Owner sends message<br><br>**Automated Messages**:<br>- System sends automated message templates at booking milestones<br>- 7 days before check-in: "Looking forward to your stay" with preparation tips<br>- 1 day before check-in: Check-in instructions with address and contact<br>- On check-in day: Welcome message with WiFi password and house rules<br>- After check-out: Thank you message with review request link<br>- Owner can enable/disable automated messages in settings<br>- Owner can customize automated message templates<br><br>**Bulk Messaging**:<br>- Owner selects multiple upcoming bookings<br>- Owner composes announcement (e.g., "Construction noise expected tomorrow")<br>- System sends to all selected guests<br>- Each guest receives personalized email (Dear [Name])<br><br>**Archive Conversation**:<br>- Owner clicks "Archive" on completed booking thread<br>- System moves thread to archived folder<br>- Thread removed from main inbox<br>- Owner can view archived threads via filter<br><br>**Flag for Follow-Up**:<br>- Owner flags important conversation<br>- Flagged threads appear at top of inbox<br>- Useful for pending questions or special requests<br>- Owner unflags when resolved<br><br>**Guest Inquiry (Pre-Booking)**:<br>- Guest not yet booked, sends inquiry via property page<br>- System creates inquiry thread (no booking reference)<br>- Owner responds to inquiry<br>- If guest books: System links inquiry thread to new booking<br><br>**Attach Media**:<br>- Owner attaches photos (room views, directions, facilities)<br>- Owner attaches documents (house rules PDF, local guide)<br>- System validates file size (<10MB) and format<br>- System uploads to Cloud Storage<br>- File link included in email to guest<br><br>**Translation Assistance** (future enhancement):<br>- System detects guest language<br>- Owner enables translation<br>- System translates messages both directions<br>- Original and translated text both visible<br><br>**Step 10**: If Email Service delivery fails, System retries 3 times with exponential backoff, logs failure, displays warning to owner "Message saved but email delivery failed. Please try alternative contact method."

**Postconditions**: **Success**: Message sent to guest via email. Conversation stored in thread. Guest can reply. Communication logged. Owner notified of replies.<br><br>**Failure**: Message not sent. Error displayed. Draft saved. Owner can retry or use alternative contact (phone).

**Outstanding Questions**: 1. Should system support real-time chat instead of email-based messaging?<br>2. Should system detect message urgency and notify owner accordingly?<br>3. Should owner be able to schedule messages to send at specific times?<br>4. Should system provide read receipts (guest opened message)?<br>5. Should system support voice messages or video attachments?<br>6. Should guest be able to rate communication quality with owner?<br>7. Should system integrate with WhatsApp or Telegram for messaging?

---

## UC-17 - Check In Guest

**Source**: `docs/requirements/use-cases/UC-17-check-in-guest.md`

**Use Case Name**: Check In Guest

**Use Case ID**: UC-17

**Summary**: Hostel Staff verifies guest identity, confirms booking details, collects required information, assigns bed or room, and completes check-in process. System updates booking status and records check-in timestamp.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A2a: Hostel Staff (generalized actor covering Receptionist and Owner). Secondary: None

**Preconditions**: Staff authenticated with check-in permissions. Confirmed booking exists with check-in date today or past. Guest has arrived at property. System operational.

**Trigger**: Guest arrives at property and approaches reception desk, or Staff proactively checks in arriving guest

**Main Sequence**: 1. Staff navigates to check-in interface<br>2. System displays today's expected arrivals list<br>&nbsp;&nbsp;&nbsp;&nbsp;2.1. For each arrival: guest name, booking reference, room type, check-in time, status<br>&nbsp;&nbsp;&nbsp;&nbsp;2.2. List sorted by check-in time<br>3. Staff searches for guest booking<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Staff enters guest name, booking reference, or email<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. System displays matching bookings<br>4. Staff selects correct booking<br>5. System displays check-in form with booking details<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Guest information (name, email, phone, nationality)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Booking details (dates, room type, number of guests, amount paid)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. Special requests from guest<br>&nbsp;&nbsp;&nbsp;&nbsp;5.4. Payment status (fully paid, deposit paid, balance due)<br>6. Staff verifies guest identity<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Staff requests government-issued ID (passport, driver's license)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Staff visually confirms name matches booking<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Staff optionally scans or photographs ID (for records)<br>7. Staff confirms guest count and details<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Staff confirms number of guests arriving<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. If guest count differs from booking: Staff updates count or discusses with guest<br>8. Staff collects additional required information<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Emergency contact (name and phone)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Expected check-out time<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Any additional requirements (extra towels, late check-out request)<br>9. Staff reviews house rules with guest<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Quiet hours, smoking policy, common area etiquette<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Guest verbally acknowledges understanding<br>10. If dorm booking: Staff assigns specific bed<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. System displays room layout with available beds<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. Staff selects bed for guest<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. If multiple guests: Staff assigns multiple beds<br>&nbsp;&nbsp;&nbsp;&nbsp;10.4. System marks selected beds as occupied<br>11. Staff provides property information to guest<br>&nbsp;&nbsp;&nbsp;&nbsp;11.1. WiFi name and password<br>&nbsp;&nbsp;&nbsp;&nbsp;11.2. Breakfast time and location (if included)<br>&nbsp;&nbsp;&nbsp;&nbsp;11.3. Key or access card for room/locker<br>&nbsp;&nbsp;&nbsp;&nbsp;11.4. Tour of facilities (optional)<br>12. Staff processes any outstanding payment<br>&nbsp;&nbsp;&nbsp;&nbsp;12.1. If balance due: Staff collects payment (cash, card, transfer)<br>&nbsp;&nbsp;&nbsp;&nbsp;12.2. Staff marks payment as received in system<br>13. Staff completes check-in<br>14. System validates all required information collected<br>15. System updates booking status to "Checked-In"<br>16. System records check-in timestamp<br>17. System marks assigned bed/room as occupied<br>18. System displays success message "Guest checked in successfully"<br>19. Staff provides receipt or check-in confirmation to guest

**Alternative Sequences**: **Step 3**: If booking not found (wrong name, reference), Staff tries alternative search criteria, contacts guest for booking confirmation email, or manually creates booking if prepaid proof provided<br><br>**Step 5**: If booking not confirmed (still pending), Staff contacts owner/manager for approval, or proceeds with check-in if payment verified<br><br>**Step 5**: If check-in date is future, System displays warning "Early check-in - Booking scheduled for [date]", Staff can proceed if room available or offer luggage storage<br><br>**Step 7**: If guest count exceeds booking capacity, Staff discusses options: pay for additional guest, book additional bed/room, or guest finds alternative accommodation<br><br>**Step 10**: If no beds available (overbooking error), Staff apologizes, contacts manager, offers alternatives (upgrade to private room, nearby property referral, refund), escalates to owner<br><br>**Step 12**: If guest unable to pay balance, Staff contacts manager, may allow pay later with signed agreement, or refuses check-in per policy<br><br>**Step 14**: If required information missing (no ID, no emergency contact), Staff cannot complete check-in, explains requirements, guest must provide information<br><br>**Late Check-In Flow**:<br>- Guest arrives after reception closes<br>- Self check-in option: Guest receives access code via email<br>- Guest retrieves key from lockbox<br>- Booking already assigned bed/room<br>- Staff completes formal check-in next morning<br><br>**Group Check-In**:<br>- Multiple guests on single booking<br>- Staff checks in lead guest<br>- Lead guest provides information for all group members<br>- Staff assigns all beds at once<br>- Individual guests collect keys<br><br>**No-Show Update**:<br>- Guest doesn't arrive by end of check-in day<br>- Staff marks booking as "No-Show"<br>- System applies no-show policy<br>- Bed/room released for resale<br><br>**ID Verification Failure**:<br>- ID name doesn't match booking<br>- Staff contacts booking holder<br>- If authorized: Guest provides authorization letter/message<br>- If not authorized: Staff refuses check-in per security policy

**Postconditions**: **Success**: Guest checked in. Booking status updated to "Checked-In". Bed/room assigned and marked occupied. Check-in timestamp recorded. Guest has room access. WiFi credentials provided.<br><br>**Failure**: Check-in not completed. Booking status unchanged. Bed/room not assigned. Error or missing information displayed. Staff resolves issue before proceeding.

**Outstanding Questions**: 1. Should system support digital ID verification (scan passport, extract data)?<br>2. Should guest sign digital agreement on tablet during check-in?<br>3. Should system generate printed welcome sheet with WiFi, rules, local tips?<br>4. Should check-in be possible via mobile app (contactless self check-in)?<br>5. Should system collect COVID vaccination status or health declarations?<br>6. Should staff be able to check in guests remotely before arrival?<br>7. Should system support express check-in for returning guests (saved information)?

---

## UC-18 - Check Out Guest

**Source**: `docs/requirements/use-cases/UC-18-check-out-guest.md`

**Use Case Name**: Check Out Guest

**Use Case ID**: UC-18

**Summary**: Hostel Staff processes guest departure, collects room keys, checks for damages or issues, settles any outstanding charges, and completes check-out. System updates booking status, releases bed/room availability, and records check-out timestamp.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A2a: Hostel Staff (generalized actor covering Receptionist and Owner). Secondary: None

**Preconditions**: Staff authenticated with check-out permissions. Guest currently checked in (booking status "Checked-In"). Check-out date is today or past. System operational.

**Trigger**: Guest approaches reception desk to check out, or Staff proactively processes scheduled departures

**Main Sequence**: 1. Staff navigates to check-out interface<br>2. System displays today's expected departures list<br>&nbsp;&nbsp;&nbsp;&nbsp;2.1. For each departure: guest name, booking reference, room/bed number, scheduled check-out time, status<br>&nbsp;&nbsp;&nbsp;&nbsp;2.2. List sorted by check-out time<br>3. Staff searches for guest booking<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. Staff enters guest name, booking reference, or room/bed number<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. System displays matching bookings<br>4. Staff selects correct booking<br>5. System displays check-out form with stay details<br>&nbsp;&nbsp;&nbsp;&nbsp;5.1. Guest information<br>&nbsp;&nbsp;&nbsp;&nbsp;5.2. Check-in and check-out dates (actual vs scheduled)<br>&nbsp;&nbsp;&nbsp;&nbsp;5.3. Room/bed assignment<br>&nbsp;&nbsp;&nbsp;&nbsp;5.4. Total amount paid<br>&nbsp;&nbsp;&nbsp;&nbsp;5.5. Outstanding charges (if any)<br>6. Staff asks guest about stay experience<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Guest provides feedback (verbal, informal)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Staff notes any issues or compliments in system<br>7. Staff collects room key or access card<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Verifies all keys returned<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. If locker key included: checks locker emptied<br>8. Staff inspects room/bed area (if possible)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Checks for damages<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Checks for lost property<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Notes any issues in system<br>9. Staff processes outstanding charges (if any)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Extra services used (laundry, meals, tours)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Damages or losses (broken items, lost keys)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. Late check-out fees (if applicable)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.4. Staff collects payment<br>&nbsp;&nbsp;&nbsp;&nbsp;9.5. System records additional charges and payment<br>10. Staff confirms guest collected all belongings<br>11. Staff provides receipt for additional charges (if any)<br>12. Staff thanks guest and invites review<br>&nbsp;&nbsp;&nbsp;&nbsp;12.1. Staff mentions review request email coming soon<br>&nbsp;&nbsp;&nbsp;&nbsp;12.2. Staff may provide business card with property details<br>13. Staff completes check-out<br>14. System validates check-out completion<br>15. System updates booking status to "Completed"<br>16. System records check-out timestamp<br>17. System releases bed/room from occupancy<br>18. System marks bed/room as "Needs Cleaning"<br>19. System displays success message "Guest checked out successfully"<br>20. System updates availability for future bookings

**Alternative Sequences**: **Step 3**: If booking not found, Staff tries alternative search methods, checks physical registration book, or asks guest for booking confirmation<br><br>**Step 5**: If check-out date is future (early check-out), System displays warning "Early check-out - Scheduled for [date]", Staff proceeds with early check-out, no refund for unused nights per policy<br><br>**Step 7**: If guest lost room key or access card, Staff charges key replacement fee (added to Step 9 outstanding charges), issues temporary exit pass if needed<br><br>**Step 8**: If damage found, Staff documents with photos, assesses damage cost, adds to outstanding charges, may require manager approval for significant damage<br><br>**Step 9**: If guest unable to pay outstanding charges, Staff contacts manager, may accept later payment with signed agreement, or withholds final receipt until paid<br><br>**Late Check-Out Handling**:<br>- Guest requests late check-out<br>- Staff checks room availability (next booking?)<br>- If available: Staff approves (free or for fee per property policy)<br>- If not available: Staff offers luggage storage<br>- System updates scheduled check-out time<br><br>**Express Check-Out**:<br>- Guest doesn't visit reception (early morning departure)<br>- Guest leaves key in drop box<br>- Staff processes check-out when discovered<br>- System marks check-out time as actual departure<br>- No outstanding charges collection possible (charged to card if authorized)<br><br>**Extended Stay Request**:<br>- Guest wants to extend stay beyond original check-out<br>- Staff checks room/bed availability<br>- If available: Staff creates new booking or modifies existing<br>- If not available: Staff offers alternative room or refers to nearby hostels<br><br>**Lost Property Found**:<br>- Guest left belongings in room<br>- Staff logs lost property item<br>- Staff contacts guest via email/phone<br>- Arranges shipping or pickup<br>- Lost property tracked in system<br><br>**Automatic Check-Out**:<br>- Guest doesn't check out by scheduled time<br>- Staff attempts to contact guest<br>- If no response by end of day: Staff processes check-out<br>- Bed/room released for next booking<br>- Outstanding charges handled per policy<br><br>**Group Check-Out**:<br>- Multiple guests on single booking<br>- Lead guest checks out for group<br>- Staff collects all keys from group<br>- Single check-out transaction<br><br>**Dispute Resolution**:<br>- Guest disputes additional charges<br>- Staff escalates to manager<br>- Manager reviews evidence (photos, logs)<br>- Resolution reached (waive, reduce, or uphold charge)<br>- Guest checks out after resolution

**Postconditions**: **Success**: Guest checked out. Booking status updated to "Completed". Check-out timestamp recorded. Bed/room released and marked for cleaning. Outstanding charges settled. Availability updated for future bookings.<br><br>**Failure**: Check-out not completed. Booking status remains "Checked-In". Issue noted for resolution. Staff escalates to manager.

**Outstanding Questions**: 1. Should system send automated check-out confirmation email to guest?<br>2. Should guest be able to check out via mobile app (express check-out)?<br>3. Should system generate summary of stay (nights stayed, services used)?<br>4. Should staff be able to rate guest behavior (for future bookings)?<br>5. Should system automatically trigger review request email immediately after check-out?<br>6. Should system support contactless key return (smart locks)?<br>7. Should lost property images be stored in system for easy identification?

---

## UC-19 - View Booking Details

**Source**: `docs/requirements/use-cases/UC-19-view-booking-details.md`

**Use Case Name**: View Booking Details

**Use Case ID**: UC-19

**Summary**: Hostel Staff views comprehensive booking information including guest details, booking dates, room assignment, payment status, and communication history. System retrieves and displays all relevant booking data for staff reference.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A2a: Hostel Staff (generalized actor covering Receptionist and Owner). Secondary: None

**Preconditions**: Staff authenticated with booking view permissions. Booking exists for property. System operational.

**Trigger**: Staff clicks on booking from dashboard, searches for booking, or scans booking reference barcode/QR code

**Main Sequence**: 1. Staff navigates to booking search or dashboard<br>2. Staff searches for booking<br>&nbsp;&nbsp;&nbsp;&nbsp;2.1. Staff enters guest name, booking reference, email, or phone number<br>&nbsp;&nbsp;&nbsp;&nbsp;2.2. Alternatively: Staff scans booking confirmation QR code<br>3. System retrieves matching bookings<br>4. System displays search results (if multiple matches)<br>5. Staff selects specific booking<br>6. System displays comprehensive booking details page<br>7. **Guest Information Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;7.1. Full name<br>&nbsp;&nbsp;&nbsp;&nbsp;7.2. Email address<br>&nbsp;&nbsp;&nbsp;&nbsp;7.3. Phone number<br>&nbsp;&nbsp;&nbsp;&nbsp;7.4. Nationality<br>&nbsp;&nbsp;&nbsp;&nbsp;7.5. Number of past bookings at property<br>&nbsp;&nbsp;&nbsp;&nbsp;7.6. Emergency contact (if checked in)<br>8. **Booking Details Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;8.1. Booking reference number<br>&nbsp;&nbsp;&nbsp;&nbsp;8.2. Booking status (Confirmed, Checked-In, Completed, Cancelled)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.3. Booked date and time<br>&nbsp;&nbsp;&nbsp;&nbsp;8.4. Check-in date (scheduled and actual)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.5. Check-out date (scheduled and actual)<br>&nbsp;&nbsp;&nbsp;&nbsp;8.6. Number of nights<br>&nbsp;&nbsp;&nbsp;&nbsp;8.7. Number of guests<br>9. **Room/Bed Assignment Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;9.1. Room type<br>&nbsp;&nbsp;&nbsp;&nbsp;9.2. Specific room number (if assigned)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.3. Bed number(s) for dorm bookings (if assigned)<br>&nbsp;&nbsp;&nbsp;&nbsp;9.4. Special requests from guest<br>10. **Payment Information Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;10.1. Total amount<br>&nbsp;&nbsp;&nbsp;&nbsp;10.2. Amount paid<br>&nbsp;&nbsp;&nbsp;&nbsp;10.3. Outstanding balance (if any)<br>&nbsp;&nbsp;&nbsp;&nbsp;10.4. Payment method used<br>&nbsp;&nbsp;&nbsp;&nbsp;10.5. Transaction ID<br>&nbsp;&nbsp;&nbsp;&nbsp;10.6. Payment date and time<br>&nbsp;&nbsp;&nbsp;&nbsp;10.7. Refund information (if applicable)<br>11. **Communication History Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;11.1. All messages between owner and guest<br>&nbsp;&nbsp;&nbsp;&nbsp;11.2. Automated system messages sent<br>&nbsp;&nbsp;&nbsp;&nbsp;11.3. Timestamps for each communication<br>12. **Activity Timeline Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;12.1. Booking created<br>&nbsp;&nbsp;&nbsp;&nbsp;12.2. Payment received<br>&nbsp;&nbsp;&nbsp;&nbsp;12.3. Booking confirmed<br>&nbsp;&nbsp;&nbsp;&nbsp;12.4. Bed/room assigned<br>&nbsp;&nbsp;&nbsp;&nbsp;12.5. Guest checked in (with timestamp)<br>&nbsp;&nbsp;&nbsp;&nbsp;12.6. Guest checked out (with timestamp)<br>&nbsp;&nbsp;&nbsp;&nbsp;12.7. Any modifications or cancellations<br>13. **Staff Notes Section**<br>&nbsp;&nbsp;&nbsp;&nbsp;13.1. Internal notes added by staff (not visible to guest)<br>&nbsp;&nbsp;&nbsp;&nbsp;13.2. Note author and timestamp<br>14. **Quick Actions Panel**<br>&nbsp;&nbsp;&nbsp;&nbsp;14.1. Contact Guest button<br>&nbsp;&nbsp;&nbsp;&nbsp;14.2. Check In button (if confirmed and date appropriate)<br>&nbsp;&nbsp;&nbsp;&nbsp;14.3. Check Out button (if checked in)<br>&nbsp;&nbsp;&nbsp;&nbsp;14.4. Modify Booking button (if permissions allow)<br>&nbsp;&nbsp;&nbsp;&nbsp;14.5. Print Booking button<br>15. Staff reviews information as needed<br>16. Staff performs action from quick actions or navigates elsewhere

**Alternative Sequences**: **Step 3**: If no bookings found, System displays "No bookings found matching your search" with suggestions to try different search criteria<br><br>**Step 3**: If search is ambiguous (common name), System displays list of all matches with key distinguishing details (dates, booking reference) for staff to select correct booking<br><br>**QR Code Scan Flow**:<br>- Staff uses device camera or barcode scanner<br>- Scans booking confirmation QR code (from guest's email or phone)<br>- System decodes QR code to extract booking reference<br>- System retrieves booking directly (skips search results)<br>- System displays booking details<br><br>**Print Booking**:<br>- Staff clicks "Print Booking" button<br>- System generates printable booking summary<br>- Includes: guest info, dates, room assignment, payment status, QR code<br>- Staff prints for physical records or guest copy<br><br>**Add Staff Note**:<br>- Staff clicks "Add Note" button<br>- Staff enters note text (e.g., "Guest requested quiet room", "VIP - owner's friend")<br>- Staff saves note<br>- System adds note to booking with staff name and timestamp<br>- Note visible to all staff, not to guest<br><br>**Contact Guest from Details**:<br>- Staff clicks "Contact Guest" button<br>- System opens message compose interface<br>- Guest email pre-filled<br>- Booking context automatically included<br>- Staff sends message (continues to UC-16)<br><br>**Quick Check-In from Details**:<br>- Staff viewing booking on check-in day<br>- Guest arrives at desk<br>- Staff clicks "Check In" button from details page<br>- System navigates to check-in form with data pre-filled (continues to UC-17)<br><br>**Mobile View Optimization**:<br>- Staff viewing on mobile device<br>- System displays responsive layout<br>- Collapsible sections to reduce scrolling<br>- Key information prioritized at top<br>- Touch-friendly action buttons<br><br>**Booking History for Repeat Guests**:<br>- System detects guest has previous bookings<br>- "View Past Bookings" link displayed<br>- Staff clicks to see guest's booking history<br>- Useful for recognizing VIP guests or addressing repeat issues

**Postconditions**: **Success**: Staff views complete booking information. Staff can take appropriate action based on information. Booking data remains unchanged (read-only unless action taken).<br><br>**Failure**: Error message if booking not accessible. Partial data displayed if available. Staff can retry or contact support.

**Outstanding Questions**: 1. Should staff be able to export individual booking details to PDF?<br>2. Should system highlight important information (special requests, VIP status)?<br>3. Should guest photo (from profile) be displayed for easier identification?<br>4. Should system show other guests in same room (for dorm awareness)?<br>5. Should staff be able to flag bookings for attention/follow-up?<br>6. Should system display estimated arrival time (from booking or communication)?<br>7. Should booking details include link to guest's full profile/history?

---

## UC-20 - Manage User Accounts

**Source**: `docs/requirements/use-cases/UC-20-manage-user-accounts.md`

**Use Case Name**: Manage User Accounts

**Use Case ID**: UC-20

**Summary**: Platform Administrator views all user accounts, suspends or deactivates accounts, resets passwords, verifies properties, and resolves account issues. System sends notifications via Email Service.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A0: Platform Administrator. Secondary: A5: Email Service

**Preconditions**: Administrator authenticated with admin permissions. System operational. Email Service available.

**Trigger**: Administrator clicks "User Management" in admin dashboard, or receives support ticket about account issue

**Main Sequence**: 1. Administrator navigates to user management page<br>2. System displays user list with filters (All, Travelers, Owners, Staff, Suspended)<br>3. Administrator searches for user (name, email, ID)<br>4. System displays user profile with: account details, properties owned, booking history, account status, login activity<br>5. Administrator performs action: view details, suspend account, reset password, verify property, delete account, or send notification<br>6. System executes action and logs activity<br>7. System sends notification email to user via Email Service (if applicable)<br>8. System displays confirmation message

**Alternative Sequences**: **Suspend Account**: Admin suspends for policy violation, System blocks login, sends email notification<br>**Reset Password**: Admin generates temporary password, sends to user email, requires change on next login<br>**Verify Property**: Admin approves property registration, updates status to Active, notifies owner<br>**Delete Account**: Admin confirms, System anonymizes data (GDPR), removes PII, retains transactional records

**Postconditions**: **Success**: User account updated. Notification sent. Action logged. Changes reflected system-wide.<br>**Failure**: Error displayed. Action not applied. Admin can retry.

**Outstanding Questions**: 1. Should system support bulk account operations?<br>2. Should admins be able to impersonate users for troubleshooting?<br>3. Should automated suspension trigger for fraud detection?<br>4. What retention period for deleted account data?

---

## UC-21 - Manage Platform Subscriptions

**Source**: `docs/requirements/use-cases/UC-21-manage-platform-subscriptions.md`

**Use Case Name**: Manage Platform Subscriptions

**Use Case ID**: UC-21

**Summary**: Platform Administrator views all subscriptions, adjusts plans for support cases, processes refunds, extends trials, and resolves billing issues. System updates subscription status and feature access.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A0: Platform Administrator. Secondary: None

**Preconditions**: Administrator authenticated. System operational.

**Trigger**: Administrator clicks "Subscriptions" in admin dashboard, or receives billing support ticket

**Main Sequence**: 1. Administrator navigates to subscriptions management<br>2. System displays subscription list with filters (Active, Trial, Expired, Cancelled)<br>3. Administrator searches subscription (owner email, property name, subscription ID)<br>4. System displays subscription details: plan, billing cycle, payment history, status<br>5. Administrator performs action: upgrade/downgrade plan, extend trial, process refund, cancel subscription, waive fees<br>6. System updates subscription and feature access<br>7. System displays confirmation with changes applied

**Alternative Sequences**: **Extend Trial**: Admin adds days to trial period, updates expiration date<br>**Process Refund**: Admin initiates refund, reason documented, amount returned to payment method<br>**Waive Fees**: Admin applies credit to account, reason documented for audit<br>**Cancel Subscription**: Admin cancels with immediate effect or end of period, documented reason

**Postconditions**: **Success**: Subscription updated. Feature access adjusted. Changes logged. Owner sees updated plan.<br>**Failure**: Error displayed. Subscription unchanged. Admin can retry.

**Outstanding Questions**: 1. Should system support bulk subscription operations?<br>2. Should refunds require approval from finance team?<br>3. Should admin changes trigger automated accounting entries?<br>4. What approval workflow for large refunds?

---

## UC-22 - Configure System Settings

**Source**: `docs/requirements/use-cases/UC-22-configure-system-settings.md`

**Use Case Name**: Configure System Settings

**Use Case ID**: UC-22

**Summary**: Platform Administrator configures platform-wide settings including commission rates, payment methods, email templates, feature flags, and maintenance mode. System applies settings across platform.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A0: Platform Administrator. Secondary: None

**Preconditions**: Administrator authenticated with system config permissions. System operational.

**Trigger**: Administrator clicks "System Settings" in admin dashboard

**Main Sequence**: 1. Administrator navigates to system configuration<br>2. System displays settings categories: General, Payment, Email, Features, Maintenance<br>3. **General Settings**: Platform name, support email, default language, timezone<br>4. **Payment Settings**: Commission rates by plan, payment gateway credentials (SePay, VNPay), currency settings<br>5. **Email Settings**: SMTP configuration, email templates (booking confirmed, check-in reminder, etc.)<br>6. **Feature Flags**: Enable/disable features (analytics, messaging, reviews), A/B test configurations<br>7. **Maintenance Mode**: Schedule maintenance window, display maintenance message to users<br>8. Administrator modifies settings<br>9. System validates configuration<br>10. Administrator saves changes<br>11. System applies settings platform-wide<br>12. System displays confirmation

**Alternative Sequences**: **Maintenance Mode**: Admin enables maintenance, System displays maintenance page to users, Admin operations continue, disables when maintenance complete<br>**Email Template Edit**: Admin modifies template with variables ({{guestName}}, {{checkInDate}}), System previews template, validates variables<br>**Feature Flag Toggle**: Admin disables feature, System hides feature from all users immediately, re-enables when ready

**Postconditions**: **Success**: Settings saved and applied platform-wide. Changes effective immediately or at scheduled time.<br>**Failure**: Validation errors displayed. Settings unchanged. Admin corrects and retries.

**Outstanding Questions**: 1. Should system support configuration versioning/rollback?<br>2. Should critical settings require dual approval?<br>3. Should system test email delivery before saving SMTP config?<br>4. Should feature flags support user segmentation?

---

## UC-23 - Monitor Platform Metrics

**Source**: `docs/requirements/use-cases/UC-23-monitor-platform-metrics.md`

**Use Case Name**: Monitor Platform Metrics

**Use Case ID**: UC-23

**Summary**: Platform Administrator monitors system health, user growth, revenue metrics, booking trends, and property statistics. System aggregates platform-wide data and displays analytics dashboard.

**Dependency**: Include: None. Extend: None

**Actors**: Primary: A0: Platform Administrator. Secondary: None

**Preconditions**: Administrator authenticated. Platform has operational data. System operational.

**Trigger**: Administrator clicks "Platform Metrics" or "Dashboard" in admin interface

**Main Sequence**: 1. Administrator navigates to platform metrics dashboard<br>2. System aggregates platform-wide data<br>3. System displays metrics overview:<br>&nbsp;&nbsp;&nbsp;&nbsp;3.1. **User Metrics**: Total users, new registrations (daily/weekly/monthly), active users, user retention rate<br>&nbsp;&nbsp;&nbsp;&nbsp;3.2. **Property Metrics**: Total properties, active properties, properties by status (pending, active, suspended), average rating<br>&nbsp;&nbsp;&nbsp;&nbsp;3.3. **Booking Metrics**: Total bookings, booking volume trend, average booking value, cancellation rate<br>&nbsp;&nbsp;&nbsp;&nbsp;3.4. **Revenue Metrics**: Total revenue, commission earned, revenue by plan tier, MRR/ARR growth<br>&nbsp;&nbsp;&nbsp;&nbsp;3.5. **System Health**: Server uptime, response times, error rates, API call volume<br>4. Administrator selects time period (7d, 30d, 90d, YTD, custom)<br>5. System recalculates metrics for period<br>6. Administrator drills down into specific metric<br>7. System displays detailed breakdown with charts<br>8. Administrator exports report (optional)<br>9. System generates CSV/PDF report

**Alternative Sequences**: **Real-Time Monitoring**: Admin views live dashboard with auto-refresh, System updates metrics every 60s<br>**Alert Configuration**: Admin sets thresholds (error rate >5%, revenue drop >20%), System sends email when threshold breached<br>**Comparison View**: Admin compares current period to previous period, System highlights significant changes (>10% variance)

**Postconditions**: **Success**: Admin views comprehensive platform metrics. Insights inform strategic decisions. Reports exported if requested.<br>**Failure**: Error if data aggregation fails. Partial metrics shown. Admin can retry.

**Outstanding Questions**: 1. Should system provide predictive analytics (revenue forecast)?<br>2. Should metrics dashboard support custom widget configuration?<br>3. Should system send automated weekly summary reports?<br>4. Should metrics include competitor benchmarking data?

---

## UC-24 - Cancel Booking

**Source**: `docs/requirements/use-cases/UC-24-cancel-booking.md`

**Use Case Name**: Cancel Booking

**Use Case ID**: UC-24

**Summary**: Traveler cancels confirmed booking within cancellation window. System validates cancellation policy, processes refund via Payment Gateway, notifies hostel owner via Email Service, and releases inventory.

**Dependency**: Include: None. Extend: UC-05 (Book Accommodation)

**Actors**: Primary: A3: Traveler. Secondary: A4: Payment Gateway, A5: Email Service

**Preconditions**: Traveler authenticated. Booking exists with status "Confirmed". System operational. Payment Gateway and Email Service available.

**Trigger**: Traveler clicks "Cancel Booking" button from booking details page (UC-06)

**Main Sequence**: 1. Traveler navigates to booking details<br>2. System displays booking with cancellation option<br>3. Traveler clicks "Cancel Booking"<br>4. System retrieves cancellation policy for booking<br>5. System calculates refund amount based on policy and timing<br>6. System displays cancellation confirmation screen<br>&nbsp;&nbsp;&nbsp;&nbsp;6.1. Booking details summary<br>&nbsp;&nbsp;&nbsp;&nbsp;6.2. Cancellation policy terms<br>&nbsp;&nbsp;&nbsp;&nbsp;6.3. Refund amount breakdown (total paid - cancellation fee = refund)<br>&nbsp;&nbsp;&nbsp;&nbsp;6.4. Processing timeline (7-14 business days)<br>7. Traveler selects cancellation reason (optional dropdown)<br>8. Traveler confirms cancellation<br>9. System validates cancellation eligibility<br>10. System updates booking status to "Cancelled"<br>11. System records cancellation timestamp and reason<br>12. System initiates refund process via Payment Gateway<br>13. Payment Gateway processes refund to original payment method<br>14. Payment Gateway confirms refund initiated<br>15. System releases bed/room from occupancy<br>16. System updates availability for future bookings<br>17. System sends cancellation confirmation email to traveler via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;17.1. Email includes: cancellation confirmation, refund amount, processing timeline, booking reference<br>18. System sends cancellation notification to owner via Email Service<br>&nbsp;&nbsp;&nbsp;&nbsp;18.1. Email includes: guest name, booking details, cancellation date, released inventory<br>19. System displays success message "Booking cancelled successfully. Refund will be processed within 7-14 business days."

**Alternative Sequences**: **Outside Cancellation Window** (>24 hours after booking):<br>- Step 5: System calculates zero refund per policy<br>- Step 6: Shows "No refund available - outside cancellation window"<br>- Traveler can proceed with cancellation (no refund) or keep booking<br>- If cancels: No refund processed, room released, notifications sent<br><br>**Partial Refund Scenario** (group booking, partial cancel):<br>- Traveler cancels portion of group booking<br>- System calculates pro-rated refund<br>- Updates booking to reduced guest count<br>- Processes partial refund<br><br>**Payment Gateway Refund Failure**:<br>- Step 13: Payment Gateway returns error<br>- System logs failure<br>- System displays "Booking cancelled but refund processing failed. Support team notified."<br>- Support team manually processes refund<br>- System updates when refund completed<br><br>**Already Checked In**:<br>- Step 9: Validation fails - guest already checked in<br>- System displays "Cannot cancel after check-in. Please contact property for early check-out."<br>- Directs traveler to contact owner<br><br>**Within 24h of Check-In**:<br>- System displays warning "Cancellation within 24h may result in fees per property policy"<br>- Shows adjusted refund amount (cancellation fee applied)<br>- Traveler proceeds or keeps booking<br><br>**Email Delivery Failure**:<br>- Step 17/18: Email Service returns error<br>- System retries delivery 3 times<br>- If all fail: System logs error, cancellation still valid<br>- Support team manually notifies parties<br><br>**Cancellation by Owner/Admin**:<br>- Owner cancels booking (overbooking, property issue)<br>- Full refund issued regardless of policy<br>- Profuse apology email sent<br>- System may offer compensation credit

**Postconditions**: **Success**: Booking cancelled. Refund processed or scheduled. Inventory released. All parties notified. Cancellation logged.<br><br>**Failure**: Cancellation blocked (validation failed). Error message displayed with reason. Booking remains active.

**Outstanding Questions**: 1. Should system offer rebooking instead of full cancellation?<br>2. Should travelers be able to cancel via email/SMS?<br>3. Should system allow cancellation without login (secure link)?<br>4. Should refund go to platform credit vs original payment method?<br>5. Should system provide cancellation protection insurance option?<br>6. What happens to loyalty points earned from cancelled bookings?<br>7. Should system blacklist users with high cancellation rates?

---
