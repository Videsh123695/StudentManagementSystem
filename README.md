# StudentManagementSystem
"This is the basic Student management System which is required for any school or college to store the information digitally where we can  store our organization's people information and retrieve them when required"

import javax.swing.*;
import java.awt.event.*;
import java.sql.*;

public class ATMProject {

    // Database Connection
    static class DB {
        public static Connection getCon() {
            try {
                Class.forName("com.mysql.cj.jdbc.Driver");
                return DriverManager.getConnection(
                        "jdbc:mysql://localhost:3036/atmDB", "root", "password@2025");
            } catch (Exception e) {
                e.printStackTrace();
                return null;
            }
        }
    }

    // Account info store karne ke liye
    static int loggedAccNo = 0;

    public static void main(String[] args) {
        JFrame frame = new JFrame("ATM Login");

        JLabel label1 = new JLabel("Account Number:");
        label1.setBounds(30, 40, 120, 30);
        JTextField accField = new JTextField();
        accField.setBounds(160, 40, 150, 30);

        JLabel label2 = new JLabel("PIN:");
        label2.setBounds(30, 80, 120, 30);
        JPasswordField pinField = new JPasswordField();
        pinField.setBounds(160, 80, 150, 30);

        JButton loginBtn = new JButton("Login");
        loginBtn.setBounds(120, 130, 100, 30);

        loginBtn.addActionListener(e -> {
            int acc = Integer.parseInt(accField.getText());
            int pin = Integer.parseInt(new String(pinField.getPassword()));
            if (validateUser(acc, pin)) {
                loggedAccNo = acc;
                JOptionPane.showMessageDialog(frame, "Login Successful!");
                frame.dispose();
                openATMWindow();
            } else {
                JOptionPane.showMessageDialog(frame, "Invalid Account or PIN!");
            }
        });

        frame.add(label1);
        frame.add(accField);
        frame.add(label2);
        frame.add(pinField);
        frame.add(loginBtn);

        frame.setSize(350, 230);
        frame.setLayout(null);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setVisible(true);
    }

    static boolean validateUser(int acc, int pin) {
        try (Connection con = DB.getCon()) {
            PreparedStatement ps = con.prepareStatement(
                    "SELECT * FROM accounts WHERE acc_no=? AND pin=?");
            ps.setInt(1, acc);
            ps.setInt(2, pin);
            return ps.executeQuery().next();
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    static double checkBalance(int acc) {
        try (Connection con = DB.getCon()) {
            PreparedStatement ps = con.prepareStatement(
                    "SELECT balance FROM accounts WHERE acc_no=?");
            ps.setInt(1, acc);
            ResultSet rs = ps.executeQuery();
            if (rs.next()) return rs.getDouble(1);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return 0;
    }

    static void deposit(int acc, double amt) {
        try (Connection con = DB.getCon()) {
            PreparedStatement ps = con.prepareStatement(
                    "UPDATE accounts SET balance = balance + ? WHERE acc_no=?");
            ps.setDouble(1, amt);
            ps.setInt(2, acc);
            ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void withdraw(int acc, double amt) {
        double bal = checkBalance(acc);
        if (amt > bal) {
            JOptionPane.showMessageDialog(null, "Insufficient Balance!");
            return;
        }
        try (Connection con = DB.getCon()) {
            PreparedStatement ps = con.prepareStatement(
                    "UPDATE accounts SET balance = balance - ? WHERE acc_no=?");
            ps.setDouble(1, amt);
            ps.setInt(2, acc);
            ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void openATMWindow() {
        JFrame atmFrame = new JFrame("ATM Interface");

        JButton checkBtn = new JButton("Check Balance");
        checkBtn.setBounds(100, 40, 150, 30);
        checkBtn.addActionListener(e -> {
            double bal = checkBalance(loggedAccNo);
            JOptionPane.showMessageDialog(atmFrame, "Current Balance: ₹" + bal);
        });

        JButton depositBtn = new JButton("Deposit");
        depositBtn.setBounds(100, 90, 150, 30);
        depositBtn.addActionListener(e -> {
            String input = JOptionPane.showInputDialog("Enter amount to deposit:");
            if (input != null) {
                double amt = Double.parseDouble(input);
                deposit(loggedAccNo, amt);
                JOptionPane.showMessageDialog(atmFrame, "Deposited ₹" + amt);
            }
        });

        JButton withdrawBtn = new JButton("Withdraw");
        withdrawBtn.setBounds(100, 140, 150, 30);
        withdrawBtn.addActionListener(e -> {
            String input = JOptionPane.showInputDialog("Enter amount to withdraw:");
            if (input != null) {
                double amt = Double.parseDouble(input);
                withdraw(loggedAccNo, amt);
                JOptionPane.showMessageDialog(atmFrame, "Withdrawn ₹" + amt);
            }
        });

        atmFrame.add(checkBtn);
        atmFrame.add(depositBtn);
        atmFrame.add(withdrawBtn);
        atmFrame.setSize(400, 250);
        atmFrame.setLayout(null);
        atmFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        atmFrame.setVisible(true);
    }
}
