```html
Boston Black Entrepreneurs Appointment Scheduler

Let's break this down into manageable pieces:
1. Key MVP Features

User registration/login for business owners and clients
Business profile creation with services, hours, and location
Appointment booking system with calendar view
Service management for creating/editing service offerings
Client management for tracking client information
Email notifications for appointment confirmations and reminders
Simple analytics showing booking trends and popular services
Mobile-responsive design for on-the-go access

2. User Stories
Business Owner Stories

As a Black barber in Roxbury, I want to display my available time slots so clients can book when I'm free
As a natural hair stylist in Dorchester, I need to specify how long each service takes so my schedule isn't overbooked
As a realtor in Mattapan, I want to confirm appointments before they're finalized to ensure I'm prepared
As a consultant in Jamaica Plain, I need to see my weekly schedule at a glance to plan my workload
As a Boston-based lawyer, I want to receive notifications when clients book or cancel appointments

Client Stories

As a client, I want to find Black-owned businesses near me in Boston by service type
As a client, I want to book appointments with my favorite stylist without calling the shop
As a client, I want to receive reminders about upcoming appointments
As a client, I need to easily reschedule or cancel if my plans change
As a client, I want to view my appointment history and easily rebook past services

3. Data Model
Businesses Table

business_id (primary key)
owner_id (foreign key to Users)
business_name
business_type (barber, stylist, etc.)
description
address
neighborhood (Roxbury, Dorchester, etc.)
phone
email
website
social_media_links
created_at
updated_at

Services Table

service_id (primary key)
business_id (foreign key)
service_name
description
duration (minutes)
price
is_active

Business Hours Table

hours_id (primary key)
business_id (foreign key)
day_of_week
open_time
close_time
is_closed (for days off)

Users Table

user_id (primary key)
email
password_hash
full_name
phone
user_type (business_owner/client)
created_at

Appointments Table

appointment_id (primary key)
business_id (foreign key)
client_id (foreign key to Users)
service_id (foreign key)
start_time
end_time
status (confirmed/pending/canceled/completed)
notes
created_at
updated_at

4. Basic Workflow

Business Owner Onboarding:

Register account
Create business profile
Add services with pricing and duration
Set regular business hours and availability


Client Booking Flow:

Register/login (or book as guest)
Browse/search Black-owned businesses in Boston
Select business and desired service
View available time slots
Select appointment time
Confirm booking details
Receive confirmation


Appointment Management:

Business owners get notifications of new bookings
Confirm or request changes to bookings
Both parties receive reminders before appointments
Post-appointment feedback option



5. Frontend Technical Considerations (HTML, Alpine.js, TailwindCSS)

Alpine.js for Interactivity:

Use for dynamic calendar views and time slot selection
Handle form validations without page refreshes
Create responsive dropdown menus for service selection
Implement modal dialogs for confirmations


TailwindCSS for Styling:

Create a mobile-first responsive design
Use a color scheme that celebrates Black culture and Boston heritage
Implement accessible components for all users
Create consistent card designs for business listings


Component Structure:

Calendar component for viewing available slots
Business profile cards for browsing
Service selection interface
Booking confirmation flows
User dashboard layouts for both clients and business owners



6. Backend Technical Considerations (Python and Supabase)

Python Backend:

Use FastAPI or Flask for building API endpoints
Implement JWT authentication for secure login
Create background jobs for sending reminders (using something like Celery)
Add validation logic for appointment conflicts


Supabase Integration:

Set up tables based on the data model above
Use Row Level Security (RLS) to protect business and client data
Implement real-time subscriptions for instant booking notifications
Create database functions for recurring availability generation
Set up authentication with Supabase Auth


API Endpoints Needed:

User registration/authentication
Business profile CRUD operations
Service management
Availability calculation
Appointment booking/modification
Notification system
                
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Boston Black Entrepreneurs - Dashboard</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js" defer></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" integrity="sha384-34aJ5JrghjZI0vJnm1N8piLm9G8J9RfhPTfTe46tQJjnC9Ly9fhcJL1A8w9E" crossorigin="anonymous">
</head>
<body class="bg-gray-100">

  <!-- Header -->
  <header class="bg-black text-white py-4">
    <div class="container mx-auto flex justify-between items-center">
      <h1 class="text-2xl font-bold">BBE Appointment Scheduler</h1>
      <nav>
        <a href="#" class="hover:text-gray-300 px-3">Dashboard</a>
        <a href="#" class="hover:text-gray-300 px-3">Bookings</a>
        <a href="#" class="hover:text-gray-300 px-3">Logout</a>
      </nav>
    </div>
  </header>

  <!-- Main Content -->
  <main class="container mx-auto py-6" x-data="dashboardContent()">
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">

      <!-- Sidebar -->
      <div class="md:col-span-1">
        <div class="bg-white rounded-lg shadow-md p-4">
          <h2 class="font-bold text-xl mb-4">Your Profile</h2>
          <img src="https://via.placeholder.com/150" alt="Profile Picture" class="rounded-full mx-auto mb-3">
          <p class="text-center font-medium" x-text="profile.fullName"></p>
          <p class="text-center text-gray-600 text-sm" x-text="profile.email"></p>
          <hr class="my-4">
          <ul class="space-y-2">
            <li><a href="#" @click.prevent="activeTab = 'profile'" class="block hover:bg-gray-100 rounded-md p-2"><i class="fas fa-user mr-2"></i> Edit Profile</a></li>
            <li><a href="#" @click.prevent="activeTab = 'business'" class="block hover:bg-gray-100 rounded-md p-2"><i class="fas fa-store mr-2"></i> Business Profile</a></li>
            <li><a href="#" @click.prevent="activeTab = 'services'" class="block hover:bg-gray-100 rounded-md p-2"><i class="fas fa-cut mr-2"></i> Services</a></li>
            <li><a href="#" @click.prevent="activeTab = 'hours'" class="block hover:bg-gray-100 rounded-md p-2"><i class="fas fa-clock mr-2"></i> Business Hours</a></li>
            <li><a href="#" @click.prevent="activeTab = 'analytics'" class="block hover:bg-gray-100 rounded-md p-2"><i class="fas fa-chart-bar mr-2"></i> Analytics</a></li>
          </ul>
        </div>
      </div>

      <!-- Content Area -->
      <div class="md:col-span-2">
        <!-- Tab Navigation -->
        <div class="mb-4">
          <nav class="flex space-x-4">
            <button @click="activeTab = 'profile'" :class="{'text-black': activeTab === 'profile', 'text-gray-600': activeTab !== 'profile'}" class="focus:outline-none">Edit Profile</button>
            <button @click="activeTab = 'business'" :class="{'text-black': activeTab === 'business', 'text-gray-600': activeTab !== 'business'}" class="focus:outline-none">Business Profile</button>
            <button @click="activeTab = 'services'" :class="{'text-black': activeTab === 'services', 'text-gray-600': activeTab !== 'services'}" class="focus:outline-none">Services</button>
             <button @click="activeTab = 'hours'" :class="{'text-black': activeTab === 'hours', 'text-gray-600': activeTab !== 'hours'}" class="focus:outline-none">Business Hours</button>
            <button @click="activeTab = 'analytics'" :class="{'text-black': activeTab === 'analytics', 'text-gray-600': activeTab !== 'analytics'}" class="focus:outline-none">Analytics</button>
          </nav>
        </div>

        <!-- Tab Content -->
        <div x-data="{ activeTab: 'profile' }">
        
          <!-- Edit Profile Tab -->
          <div x-show="activeTab === 'profile'" class="bg-white rounded-lg shadow-md p-6">
            <h2 class="font-bold text-2xl mb-6">Edit Profile</h2>
            
            <form @submit.prevent="saveProfile">
              <div class="grid grid-cols-1 gap-4 md:grid-cols-2">
                <div>
                  <label for="fullName" class="block text-sm font-medium text-gray-700 mb-1">Full Name</label>
                  <input type="text" id="fullName" x-model="profile.fullName" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
                <div>
                  <label for="email" class="block text-sm font-medium text-gray-700 mb-1">Email Address</label>
                  <input type="email" id="email" x-model="profile.email" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
                <div>
                  <label for="phone" class="block text-sm font-medium text-gray-700 mb-1">Phone Number</label>
                  <input type="tel" id="phone" x-model="profile.phone" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
                <div>
                  <label for="address" class="block text-sm font-medium text-gray-700 mb-1">Address</label>
                  <input type="text" id="address" x-model="profile.address" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
              </div>
              
              <div class="mt-6">
                <button type="submit" class="px-6 py-3 bg-black text-white rounded-md hover:bg-gray-800">Save Profile</button>
              </div>
            </form>
          </div>
          
          <!-- Business Profile Tab -->
          <div x-show="activeTab === 'business'" class="bg-white rounded-lg shadow-md p-6">
            <h2 class="font-bold text-2xl mb-6">Business Profile</h2>
            
            <form @submit.prevent="saveBusinessProfile">
              <div class="grid grid-cols-1 gap-4 md:grid-cols-2">
                <div>
                  <label for="businessName" class="block text-sm font-medium text-gray-700 mb-1">Business Name</label>
                  <input type="text" id="businessName" x-model="businessProfile.businessName" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
                <div>
                  <label for="businessType" class="block text-sm font-medium text-gray-700 mb-1">Business Type</label>
                  <select id="businessType" x-model="businessProfile.businessType" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                    <option value="barber">Barber</option>
                    <option value="stylist">Stylist</option>
                    <option value="realtor">Realtor</option>
                    <option value="consultant">Consultant</option>
                    <option value="lawyer">Lawyer</option>
                  </select>
                </div>
                <div class="md:col-span-2">
                  <label for="description" class="block text-sm font-medium text-gray-700 mb-1">Description</label>
                  <textarea id="description" x-model="businessProfile.description" rows="3" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent"></textarea>
                </div>
                <div>
                  <label for="address" class="block text-sm font-medium text-gray-700 mb-1">Address</label>
                  <input type="text" id="address" x-model="businessProfile.address" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
                <div>
                  <label for="neighborhood" class="block text-sm font-medium text-gray-700 mb-1">Neighborhood</label>
                  <select id="neighborhood" x-model="businessProfile.neighborhood" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                    <option value="roxbury">Roxbury</option>
                    <option value="dorchester">Dorchester</option>
                    <option value="mattapan">Mattapan</option>
                    <option value="jamaica_plain">Jamaica Plain</option>
                    <option value="downtown">Downtown</option>
                  </select>
                </div>
                <div>
                  <label for="phone" class="block text-sm font-medium text-gray-700 mb-1">Phone Number</label>
                  <input type="tel" id="phone" x-model="businessProfile.phone" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
                <div>
                  <label for="email" class="block text-sm font-medium text-gray-700 mb-1">Email Address</label>
                  <input type="email" id="email" x-model="businessProfile.email" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
                <div>
                  <label for="website" class="block text-sm font-medium text-gray-700 mb-1">Website (optional)</label>
                  <input type="url" id="website" x-model="businessProfile.website" placeholder="https://yourwebsite.com" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
                <div>
                  <label for="social" class="block text-sm font-medium text-gray-700 mb-1">Social Media (optional)</label>
                  <input type="text" id="social" x-model="businessProfile.social" placeholder="Instagram, Facebook, etc." class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                </div>
              </div>
              
              <div class="mt-6">
                <button type="submit" class="px-6 py-3 bg-black text-white rounded-md hover:bg-gray-800">Save Business Profile</button>
              </div>
            </form>
          </div>
          
          <!-- Manage Services Tab -->
          <div x-show="$parent.activeTab === 'services'" class="bg-white rounded-lg shadow-md p-6">
            <h2 class="font-bold text-2xl mb-6">Manage Services</h2>
            
            <!-- Services List -->
            <div class="mb-8">
              <h3 class="font-bold text-lg mb-3">Your Services</h3>
              <div class="space-y-3">
                <template x-for="(service, index) in services" :key="index">
                  <div class="border border-gray-200 rounded-lg p-4 flex justify-between items-center">
                    <div>
                      <h4 class="font-medium" x-text="service.name"></h4>
                      <p class="text-gray-600 text-sm" x-text="`$${service.price} · ${service.duration} minutes`"></p>
                    </div>
                    <div class="flex space-x-2">
                      <button 
                        @click="editService(index)" 
                        class="p-2 bg-gray-100 hover:bg-gray-200 rounded-md"
                      >
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" />
                        </svg>
                      </button>
                      <button 
                        @click="deleteService(index)" 
                        class="p-2 bg-red-100 hover:bg-red-200 text-red-600 rounded-md"
                      >
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                        </svg>
                      </button>
                    </div>
                  </div>
                </template>
              </div>
            </div>
            
            <!-- Add/Edit Service Form -->
            <div class="bg-gray-50 p-4 rounded-lg">
              <h3 class="font-bold text-lg mb-4" x-text="editingServiceIndex !== null ? 'Edit Service' : 'Add New Service'"></h3>
              <form @submit.prevent="saveService">
                <div class="grid grid-cols-1 gap-4 md:grid-cols-3">
                  <div class="md:col-span-3">
                    <label for="serviceName" class="block text-sm font-medium text-gray-700 mb-1">Service Name</label>
                    <input type="text" id="serviceName" x-model="currentService.name" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                  </div>
                  <div>
                    <label for="servicePrice" class="block text-sm font-medium text-gray-700 mb-1">Price ($)</label>
                    <input type="number" id="servicePrice" x-model="currentService.price" min="0" step="0.01" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                  </div>
                  <div>
                    <label for="serviceDuration" class="block text-sm font-medium text-gray-700 mb-1">Duration (minutes)</label>
                    <input type="number" id="serviceDuration" x-model="currentService.duration" min="5" step="5" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                  </div>
                  <div>
                    <label for="serviceActive" class="block text-sm font-medium text-gray-700 mb-1">Status</label>
                    <select id="serviceActive" x-model="currentService.isActive" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent">
                      <option :value="true">Active</option>
                      <option :value="false">Hidden</option>
                    </select>
                  </div>
                  <div class="md:col-span-3">
                    <label for="serviceDescription" class="block text-sm font-medium text-gray-700 mb-1">Description (optional)</label>
                    <textarea id="serviceDescription" x-model="currentService.description" rows="2" class="w-full p-3 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent"></textarea>
                  </div>
                </div>
                
                <div class="mt-4 flex space-x-3">
                  <button type="submit" class="px-4 py-2 bg-black text-white rounded-md hover:bg-gray-800">
                    <span x-text="editingServiceIndex !== null ? 'Update Service' : 'Add Service'"></span>
                  </button>
                  <button 
                    type="button" 
                    @click="resetServiceForm" 
                    class="px-4 py-2 bg-gray-100 hover:bg-gray-200 rounded-md"
                  >
                    Cancel
                  </button>
                </div>
              </form>
            </div>
          </div>
          
          <!-- Business Hours Tab -->
          <div x-show="$parent.activeTab === 'hours'" class="bg-white rounded-lg shadow-md p-6">
            <h2 class="font-bold text-2xl mb-6">Business Hours</h2>
            
            <div class="space-y-4">
              <template x-for="(day, index) in businessHours" :key="day.day">
                <div class="border border-gray-200 rounded-lg p-4">
                  <div class="flex justify-between items-center">
                    <h3 class="font-medium text-lg" x-text="day.day"></h3>
                    <label class="flex items-center">
                      <input type="checkbox" x-model="day.isClosed" class="rounded border-gray-300 text-black focus:ring-black">
                      <span class="ml-2 text-sm" x-text="day.isClosed ? 'Closed' : 'Open'"></span>
                    </label>
                  </div>
                  
                  <div x-show="!day.isClosed" class="mt-3 grid grid-cols-1 sm:grid-cols-2 gap-4">
                    <div>
                      <label :for="`openTime-${index}`" class="block text-sm font-medium text-gray-700 mb-1">Opening Time</label>
                      <input 
                        type="time" 
                        :id="`openTime-${index}`" 
                        x-model="day.openTime" 
                        class="w-full p-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent"
                      >
                    </div>
                    <div>
                      <label :for="`closeTime-${index}`" class="block text-sm font-medium text-gray-700 mb-1">Closing Time</label>
                      <input 
                        type="time" 
                        :id="`closeTime-${index}`" 
                        x-model="day.closeTime" 
                        class="w-full p-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-black focus:border-transparent"
                      >
                    </div>
                  </div>
                </div>
              </template>
            </div>
            
            <div class="mt-6">
              <button @click="saveBusinessHours" class="px-6 py-3 bg-black text-white rounded-md hover:bg-gray-800">Save Business Hours</button>
            </div>
          </div>
          
          <!-- Analytics Tab -->
          <div x-show="$parent.activeTab === 'analytics'" class="bg-white rounded-lg shadow-md p-6">
            <h2 class="font-bold text-2xl mb-6">Analytics</h2>
            
            <!-- Summary Cards -->
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 mb-8">
              <div class="bg-gray-50 p-4 rounded-lg">
                <h3 class="text-sm text-gray-500 font-medium">Total Appointments</h3>
                <p class="text-3xl font-bold mt-1">47</p>
                <p class="text-sm text-green-600 mt-2">↑ 12% from last month</p>
              </div>
              <div class="bg-gray-50 p-4 rounded-lg">
                <h3 class="text-sm text-gray-500 font-medium">Revenue</h3>
                <p class="text-3xl font-bold mt-1">$1,240</p>
                <p class="text-sm text-green-600 mt-2">↑ 8% from last month</p>
              </div>
              <div class="bg-gray-50 p-4 rounded-lg">
                <h3 class="text-sm text-gray-500 font-medium">New Clients</h3>
                <p class="text-3xl font-bold mt-1">9</p>
                <p class="text-sm text-green-600 mt-2">↑ 3 more than last month</p>
              </div>
            </div>
            
            <!-- Popular Services Chart -->
            <div class="mb-8">
              <h3 class="font-bold text-lg mb-4">Popular Services</h3>
              <div class="bg-gray-50 p-4 rounded-lg">
                <div class="space-y-3">
                  <div>
                    <div class="flex justify-between text-sm">
                      <span>Regular Haircut</span>
                      <span>42%</span>
                    </div>
                    <div class="w-full bg-gray-200 rounded-full h-2.5 mt-1">
                      <div class="bg-black h-2.5 rounded-full" style="width: 42%"></div>
                    </div>
                  </div>
                  <div>
                    <div class="flex justify-between text-sm">
                      <span>Beard Trim</span>
                      <span>28%</span>
                    </div>
                    <div class="w-full bg-gray-200 rounded-full h-2.5 mt-1">
                      <div class="bg-black h-2.5 rounded-full" style="width: 28%"></div>
                    </div>
                  </div>
                  <div>
                    <div class="flex justify-between text-sm">
                      <span>Full Service (Cut & Beard)</span>
                      <span>30%</span>
                    </div>
                    <div class="w-full bg-gray-200 rounded-full h-2.5 mt-1">
                      <div class="bg-black h-2.5 rounded-full" style="width: 30%"></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- Recent Activity -->
            <div>
              <h3 class="font-bold text-lg mb-4">Recent Activity</h3>
              <div class="bg-gray-50 p-4 rounded-lg">
                <div class="space-y-4">
                  <div class="flex">
                    <div class="flex-shrink-0 mr-3">
                      <div class="w-8 h-8 bg-gray-300 rounded-full flex items-center justify-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 text-gray-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                        </svg>
                      </div>
                    </div>
                    <div>
                      <p class="text-sm font-medium">New appointment booked</p>
                      <p class="text-xs text-gray-500">Regular Haircut with Marcus Johnson</p>
                      <p class="text-xs text-gray-400 mt-1">Today, 2:30 PM</p>
                    </div>
                  </div>
                  <div class="flex">
                    <div class="flex-shrink-0 mr-3">
                      <div class="w-8 h-8 bg-gray-300 rounded-full flex items-center justify-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 text-gray-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                        </svg>
                      </div>
                    </div>
                    <div>
                      <p class="text-sm font-medium">Appointment completed</p>
                      <p class="text-xs text-gray-500">Full Service with Keisha Williams</p>
                      <p class="text-xs text-gray-400 mt-1">Yesterday, 11:00 AM</p>
                    </div>
                  </div>
                  <div class="flex">
                    <div class="flex-shrink-0 mr-3">
                      <div class="w-8 h-8 bg-gray-300 rounded-full flex items-center justify-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 text-gray-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                        </svg>
                      </div>
                    </div>
                    <div>
                      <p class="text-sm font-medium">New review received</p>
                      <p class="text-xs text-gray-500">5 stars from Marcus Johnson</p>
                      <p class="text-xs text-gray-400 mt-1">Yesterday, 4:15 PM</p>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
  
  <!-- Footer will be the same as your index.html -->
  
  <script>
    function dashboardContent() {
      return {
        // Sample data for upcoming appointments
        upcomingAppointments: [
          {
            id: 1,
            business: "Divine Cuts Barbershop - Roxbury",
            service: "Regular Haircut",
            dateTime: new Date(new Date().getTime() + 2 * 24 * 60 * 60 * 1000) // 2 days from now
          },
          {
            id: 2,
            business: "Natural Beauty Salon - Dorchester",
            service: "Natural Hair Styling",
            dateTime: new Date(new Date().getTime() + 5 * 24 * 60 * 60 * 1000) // 5 days from now
          }
        ],
        
        // Sample data for past appointments
        pastAppointments: [
          {
            id: 3,
            business: "Divine Cuts Barbershop - Roxbury",
            service: "Beard Trim",
            dateTime: new Date(new Date().getTime() - 3 * 24 * 60 * 60 * 1000), // 3 days ago
            hasReview: true
          },
          {
            id: 4,
            business: "Boston Realty Partners - Mattapan",
            service: "Property Consultation",
            dateTime: new Date(new Date().getTime() - 10 * 24 * 60 * 60 *