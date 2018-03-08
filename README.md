import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.Arrays;
import java.util.Scanner;


public class Functions {

    
    /* 
    A class used to easily transfer the results aroudn the class.
    A 2d Array was complicated to move aroudn through methods, when the size was final.
    Using reflexion also caused a lot of complications
    This essentially creates a 2d array, that can be declared without having a final size.
    Using a 2d array allows to store information for potential use in other versions or in the future.
    */
    private static class SResult{
        private static String[][] tubles;
        
        private static int getLength(){
            return tubles[0].length;
        }
        
        public static void set(int x, int y, String z){
            tubles[x][y] = z;
        }
        
        public SResult(int x, int y){
            this.tubles = new String[x][y];
        }
        
        public static void printResult(){
                System.out.println("");
            for(int i = 0; i < tubles.length; i++){
                System.out.println(Arrays.toString(tubles[i]));
                
            }
            System.out.println("");
        }
        
        public String toString(){
            return Arrays.deepToString(this.tubles);
        }
    }
    
    /*
    Initializing various global variables for future use.
    */
    private static final String url = "jdbc:postgresql://dumbo.db.elephantsql.com:5432/anjuovvx";
    private static final String username = "anjuovvx";
    private static final String password = "KjfKr1yDAecKFzZPfAX6pEx6pFDlVyur";
    private static SResult resultList;
    
    
    private static void connectDatabase(){
    
      
        try {
            Class.forName("org.postgresql.Driver");
        } catch (java.lang.ClassNotFoundException e) {
            System.out.println(e);
        }
    }    
    
    
    /*
    Welcomming text, describing possible options
    */
    private static void welcomeMsg(){
        System.out.println("You have connected to the server " + url);
        System.out.println("You have 4 options:");
        System.out.println("option \"a\": \nPrint out the names of all coaches, and the team they belong to\n");
        System.out.println("option \"b\": \nPrint out the names of all players and coaches that has won a tournament\n");
        System.out.println("option \"c\": \nPrint out the names of all teams and their amount of players");
        System.out.println("option \"d\": \nPrint out the names of all tournaments, \nthat has participating teams amount to a number specified by a secondary input\n");
        System.out.println("Enter your function, please. Type either a, b, c, or d."); 
        
      
    }
    
    /*
    The method that takes an input, and invokes the proper method  
    Currently it invokes the method printResult, to print out the result
    But The program has been made to in the future allow for using the result for ...
    */
    private static void menu(){
        
        //Try for establishing the connection
        try {
            Connection db = DriverManager.getConnection(url, username, password);

            //Input
            Scanner sc = new Scanner(System.in);
            String function = sc.nextLine();

            //getting the appropriate method
            try {
                java.lang.reflect.Method method;

                method = Functions.class.getMethod(function, Connection.class);

                //Invoking the method in a try and catch
                try {
                    method.invoke(db, db);
                    resultList.printResult();
                }
                //Catches for the invoking try:
                catch (IllegalArgumentException e) {System.out.println(e);}
                catch (IllegalAccessException e){System.out.println(e);}
            }
            //The catches for the getter
            catch (SecurityException e) {System.out.println(e);}
            catch (NoSuchMethodException e) {System.out.println(e);}
        // catcher for establishing the connection
        }catch (Exception e) {e.getCause();}
    }
  
    /*
    The method that stores the result of the SQL Query in the 2d array
    */
    private static void sqlOrder(String order, int[] attr, Connection db){
        try {
            //Executing the Query
            Statement st = db.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE,ResultSet.CONCUR_READ_ONLY);
            ResultSet rs = st.executeQuery(order);
            
            //Storing the results
            int i = 0;
            if (rs.last()) {
                //Getting the amount of data stored
                int rows = rs.getRow();
                rs.beforeFirst();
                resultList = new SResult(rows, attr.length);
                
                //Inputting the data, sorted in tubles
                while(rs.next() == true){
                    for (int n = 0; n < resultList.getLength(); n++){   
                        resultList.set(i, n, rs.getString(attr[n])) ;
                    }                    
                    i++;                    
                }
            }               
        }
        catch (Exception e) {System.out.println(e);}
    }
  

    //The different possible SQL queries
    
    public static void a(Connection db){
        String order = "SELECT People.name, CoachesFor.TeamName FROM People, CoachesFor WHERE People.email = CoachesFor.CoachEmail";    
        int[] attr = new int[]{1,2};
        sqlOrder(order, attr, db);
    }   
		
    public static void b(Connection db){
        String order = "SELECT name FROM People WHERE Email IN " +
                  "((SELECT PlayerEmail FROM PlaysFor WHERE " +
                  "(TeamName, TeamCountry, DateFormed) IN " +
                  "(SELECT DISTINCT WinnerName, TeamCountry, DateFormed FROM WINNERS)) " +
                  "UNION " +
                  "(SELECT CoachEmail FROM CoachesFor WHERE " +
                  "(TeamName, TeamCountry, DateFormed) IN " +
                  "(SELECT DISTINCT WinnerName, TeamCountry, DateFormed FROM Winners)))";
        int[] attr = new int[]{1};
        sqlOrder(order, attr, db);
    }
    
    public static void c(Connection db){
            String order = "SELECT TeamName, COUNT(PlayerEmail) FROM PlaysFor GROUP BY TeamName";
            int[] attr = new int[]{1, 2};
            sqlOrder(order, attr, db);
    }
  
    public static void d(Connection db){
        System.out.println("Input a number, please:");
        Scanner sc = new Scanner(System.in);
        String num = sc.nextLine();
        String order = "SELECT TourneyName FROM Participates GROUP BY TourneyName HAVING COUNT(TourneyName) >=" + num;	
        int[] attr = new int[]{1};
        sqlOrder(order, attr, db);
        
    }
    
    //Main
    public static void main(String[] args) {
					connectDatabase();
					welcomeMsg();
					menu();
            
      
    }
}
