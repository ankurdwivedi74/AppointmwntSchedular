package com.appschedular;

import java.sql.*;
import java.util.Scanner;

class AppointmentScheduler {
    private static final String URL = "jdbc:mysql://localhost:3306/appointments_db";
    private static final String USER = "root";
    private static final String PASSWORD = "ankur@7415#";
    private static final Scanner scanner = new Scanner(System.in);

    public static void main(String[] args) {
        System.out.println("Welcome to the Appointment Scheduler!");
        System.out.print("Are you an admin? (yes/no): ");
        String role = scanner.next();
        if (role.equalsIgnoreCase("yes")) {
            adminMenu();
        } else {
            userMenu();
        }
    }

    private static void adminMenu() {
        int choice;
        do {
            System.out.println("\nAdmin Menu:");
            System.out.println("1: View all appointments");
            System.out.println("2: Cancel an appointment");
            System.out.println("3: View available and booked slots");
            System.out.println("4: Exit");
            System.out.print("Enter your choice: ");
            choice = scanner.nextInt();
            switch (choice) {
                case 1: viewAppointments(); break;
                case 2: cancelAppointment(); break;
                case 3: viewAvailableAndBookedSlots(); break;
                case 4: System.out.println("Exiting..."); break;
                default: System.out.println("Invalid choice. Try again.");
            }
        } while (choice != 4);
    }

    private static void userMenu() {
        int choice;
        do {
            System.out.println("\nUser Menu:");
            System.out.println("1: Book an appointment");
            System.out.println("2: Exit");
            System.out.print("Enter your choice: ");
            choice = scanner.nextInt();
            switch (choice) {
                case 1: addAppointment(); break;
                case 2: System.out.println("Exiting..."); break;
                default: System.out.println("Invalid choice. Try again.");
            }
        } while (choice != 2);
    }

    private static void addAppointment() {
        System.out.print("Enter your name: ");
        String name = scanner.next();
        viewAvailableSlots();
        System.out.print("Enter slot ID to book: ");
        int slotId = scanner.nextInt();
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD);
             PreparedStatement ps = con.prepareStatement("UPDATE appointments SET isBooked = 1, name = ? WHERE id = ? AND isBooked = 0")) {
            ps.setString(1, name);
            ps.setInt(2, slotId);
            if (ps.executeUpdate() > 0) {
                System.out.println("Appointment booked successfully!");
            } else {
                System.out.println("Slot is already booked or invalid!");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void viewAppointments() {
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD);
             Statement stmt = con.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM appointments WHERE isBooked = 1")) {
            System.out.println("Booked Appointments:");
            while (rs.next()) {
                System.out.printf("ID: %d | Time: %s | Date: %s | Name: %s\n",
                        rs.getInt("id"), rs.getString("timeSlot"), rs.getString("date"), rs.getString("name"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void viewAvailableSlots() {
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD);
             Statement stmt = con.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM appointments WHERE isBooked = 0")) {
            System.out.println("Available Slots:");
            while (rs.next()) {
                System.out.printf("ID: %d | Time: %s | Date: %s\n",
                        rs.getInt("id"), rs.getString("timeSlot"), rs.getString("date"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void viewAvailableAndBookedSlots() {
        System.out.println("\nAvailable Slots:");
        viewAvailableSlots();
        System.out.println("\nBooked Slots:");
        viewAppointments();
    }

    private static void cancelAppointment() {
        viewAppointments();
        System.out.print("Enter appointment ID to cancel: ");
        int slotId = scanner.nextInt();
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD);
             PreparedStatement ps = con.prepareStatement("UPDATE appointments SET isBooked = 0, name = NULL WHERE id = ?")) {
            ps.setInt(1, slotId);
            if (ps.executeUpdate() > 0) {
                System.out.println("Appointment canceled successfully!");
            } else {
                System.out.println("Invalid appointment ID.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
