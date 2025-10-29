import java.util.*;

class InvalidWaterException extends Exception {
    public InvalidWaterException(String message) {
        super(message);
    }
}

enum ConsumerType {
    RESIDENTIAL(1.5),
    COMMERCIAL(2.5),
    INDUSTRIAL(4.0);
    final double rate;
    ConsumerType(double r) { 
    this.rate = r; 
    }
}

// 3. CLASS - Water Tank
class WaterTank {
    private String id;
    private double capacity;
    private double currentLevel; 
    public WaterTank(String id, double capacity) {
        this.id = id;
        this.capacity = capacity;
        this.currentLevel = capacity;
    }
    public void supply(double amount) throws InvalidWaterException {
        if (amount > currentLevel) {
            throw new InvalidWaterException("Insufficient water in " + id);
        }
        currentLevel -= amount;
    }    
    public void refill() {
        currentLevel = capacity;
        System.out.println("Tank " + id + " refilled!");
    }
    public String getStatus() {
        double percent = (currentLevel / capacity) * 100;
        if (percent < 10) return "Empty";
        if (percent < 40) return "Low";
        if (percent < 80) return "Medium";
        return "High";
    }    
    @Override
    public String toString() {
        return "Tank[" + id + " - " + getStatus() + ": " +                String.format("%.1f", currentLevel) + "/" + capacity + "L]";
    }
}

class Consumer {
    private String id;
    private String name;
    private ConsumerType type;
    private double totalUsage = 0;
    private ArrayList<Double> history = new ArrayList<>();    
    public Consumer(String id, String name, ConsumerType type) {
        this.id = id;
        this.name = name;
        this.type = type;
    }    
    public void addUsage(double amount) {
        history.add(amount);
        totalUsage += amount;
    }    
    public double getBill() {
        return totalUsage * type.rate;
    }
    public String getName() { return name; }
    public String getId() { return id; }
    public ConsumerType getType() { return type; }
    public double getTotalUsage() { return totalUsage; }
    public boolean hasLeak() {
        if (history.size() < 2) return false;        
        double sum = 0;
        for (int i = 0; i < history.size() - 1; i++) {
            sum += history.get(i);
        }
        double avg = sum / (history.size() - 1);
        double current = history.get(history.size() - 1);        
        if (avg == 0) return false;
        return ((current - avg) / avg) > 0.15;
    }
    @Override
    public String toString() {
        return name + " (" + id + ", " + type + ", Usage: " + 
               String.format("%.1f", totalUsage) + "L)";
    }
}
public class WaterManagementSystem {    
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        ArrayList<WaterTank> tanks = new ArrayList<>();
        HashMap<String, Consumer> consumers = new HashMap<>();
        Queue<String> zones = new LinkedList<>();        
        System.out.println("=== WATER MANAGEMENT SYSTEM ===\n");
        tanks.add(new WaterTank("TANK-001", 5000));
        tanks.add(new WaterTank("TANK-002", 3000));
        System.out.println("✓ 2 tanks added\n");
        try {
            System.out.println("--- Register 2 Consumers ---");
            for (int i = 1; i <= 2; i++) {
                System.out.print("Consumer " + i + " ID: ");
                String id = sc.nextLine();                
                System.out.print("Name: ");
                String name = sc.nextLine();                
                System.out.println("Type: 1=RESIDENTIAL, 2=COMMERCIAL, 3=INDUSTRIAL");
                System.out.print("Choose: ");
                int choice = sc.nextInt();
                sc.nextLine();                
                ConsumerType type;
                switch (choice) {
                    case 1: type = ConsumerType.RESIDENTIAL; break;
                    case 2: type = ConsumerType.COMMERCIAL; break;
                    case 3: type = ConsumerType.INDUSTRIAL; break;
                    default: type = ConsumerType.RESIDENTIAL;
                }                
                Consumer consumer = new Consumer(id, name, type);
                consumers.put(id, consumer);
                System.out.println("✓ " + name + " registered!\n");
            }            
        } catch (InputMismatchException e) {
            System.out.println("Error: Invalid input!");
            sc.nextLine();
        }
        System.out.println("\n--- Water Supply ---");
        for (Consumer c : consumers.values()) {
            try {
                System.out.print("Supply amount for " + c.getName() + " (L): ");
                double amount = sc.nextDouble();
                sc.nextLine();
                tanks.get(0).supply(amount);
                c.addUsage(amount);                
                System.out.println("✓ Supplied " + amount + "L to " + c.getName());
                if (c.hasLeak()) {
                    System.out.println("⚠ LEAK DETECTED at " + c.getName() + "!");
                }                
            } catch (InvalidWaterException e) {
                System.out.println("✗ Error: " + e.getMessage());
            } catch (InputMismatchException e) {
                System.out.println("✗ Invalid input!");
                sc.nextLine();
            }
        }
        System.out.println("\n--- Tank Status ---");
        for (WaterTank tank : tanks) {
            System.out.println(tank);
        }
        System.out.println("\n--- Billing Report ---");
        double totalRevenue = 0;        
        for (Consumer c : consumers.values()) {
            double bill = c.getBill();
            totalRevenue += bill;            
            System.out.println(c);
            System.out.println("Bill: $" + String.format("%.2f", bill));
            System.out.println("---");
        }        
        System.out.println("Total Revenue: $" + String.format("%.2f", totalRevenue));
        zones.offer("Zone-A");
        zones.offer("Zone-B");
        zones.offer("Zone-C");        
        System.out.println("\n--- Active Zones (Queue) ---");
        System.out.println("All zones: " + zones);
        System.out.println("Processing: " + zones.poll());
        System.out.println("Remaining: " + zones);
        System.out.print("\n--- Search Consumer ---");
        System.out.print("\nEnter consumer ID: ");
        String searchId = sc.nextLine();        
        if (consumers.containsKey(searchId)) {
            System.out.println("Found: " + consumers.get(searchId));
        } else {
            System.out.println("Consumer not found!");
        }
        System.out.println("\n--- Statistics ---");
        int index = 0;
        double totalUsage = 0;
        while (index < consumers.size()) {
            for (Consumer c : consumers.values()) {
                totalUsage += c.getTotalUsage();
                break;
            }
            index++;
        }
        int count = 0;
        do {
            count++;
        } while (count < consumers.size());
        System.out.println("Total consumers: " + count);
        System.out.println("Average usage: " + String.format("%.1f", totalUsage / count) + "L");
        int[] readings = {100, 150, 120, 180, 140};
        Arrays.sort(readings);
        System.out.println("\nSorted readings: " + Arrays.toString(readings));
        String message = "Water Conservation";
        System.out.println("\nMessage: " + message);
        System.out.println("Length: " + message.length());
        System.out.println("Uppercase: " + message.toUpperCase());
        System.out.println("\nRandom meter: " + Math.round(Math.random() * 100));
        System.out.println("Tank volume (π×r²×h): " + Math.PI * Math.pow(5, 2) * 10);        
        sc.close();
        System.out.println("\n=== SYSTEM COMPLETE ===");
    }
}
