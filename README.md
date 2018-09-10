# Football series
last work from python course

# Titel: Fotbollsserie Slutinlämning efter redovisning

# Author: Pernilla Patterson Irevång
# Datum: 3 augusti 2017
# Ett program som hanterar serietabellen i Premier League. Man
# kan lägga in resultat, titta på tabellen, spara till fil med namn "PremierL.txt", och avsluta

#Programmet erbjuder att antingen starta en ny säsong med
#nollade resultat som skrivs från listan i main metoden.
#eller fortsätta på den påbörjade säsongen som sparats på fil.

#Exempel på en rad i filen är:
#Watford,0,0,0,0,0,0,0 där nollorna i tur och ordning från vänster står för:
#spelade matcher, vunna matcher, oavgjorda, förlorade matcher
#vunna mål, insläppta mål och poäng 

#importerar operator module 
from operator import attrgetter, methodcaller
from os.path import exists


# En klass som beskriver ett fotbollslag.
# name = lagets namn, played = antal matcher spelade, won = antal vunna matcher, tie= antal oavgjorda matcher, lost = antal förlorade matcher
# goals_won = antal gjorda mål, goals_lost = antal insläppta mål, points = antal poäng

class Team:
    #skapar ett nytt lag
    def __init__(self, name, played, won, tie, lost, goals_won, goals_lost, points):
        self.name = name
        self.played = played
        self.won = won
        self.tie = tie
        self.lost = lost
        self.goals_won = goals_won
        self.goals_lost = goals_lost
        self.points = points

    #returnerar en stäng som beskriver laget 
    def __str__(self):
        if len(self.name)<8:
            SPACE = TAB2
        else:
            SPACE = TAB1
        team_line = self.name + SPACE + str(self.played) + TAB1 + str(self.won) + TAB1 + str(self.tie) + TAB1 + str(self.lost) + TAB1 + str(self.goals_won) + "-" + str(self.goals_lost) + TAB1 + str(self.points)
        return team_line

#-----------------en klass som beskriver en fotbollserietabell.-----------------------------  
class Team_table:
    # Skapar en tabell med de lag som finns i filen.
    # teams = en lista som innehåller alla lag
    # 
    def __init__(self, filename):
        self.teams = list()
        file = open(filename, "r")
        for line in file:
            parts = line.strip().split(",")
            ## här görs nollorna om till int för att kunna påverkas av de olika metoder
            team = Team(parts[0], int(parts[1]), int(parts[2]), int(parts[3]), int(parts[4]), int(parts[5]), int(parts[6]), int(parts[7]))
            self.teams.append(team)
        file.close()
        
    # Sparar hela bibliotekskatalogen i en fil och gör om allt till string.
    def save_table(self, filename):
        self.sort_list()
        file = open(filename, "w")
        for team in self.teams:
            file.write(team.name + "," + str(team.played) + "," + str(team.won) + "," + str(team.tie) + "," + str(team.lost) + "," + str(team.goals_won) + "," + str(team.goals_lost) + "," + str(team.points))
            file.write("\n")
        file.close()

    # går igenom hela listan med lagnamn och anropar
    # param team_name; lagnamnet som ska kollas i listan
    # metoden same_name som returnerar true om det inskrivna namnen finns i listan.
    # return team; det lag som är inskrivet och matchas i tabellen, annars returneras None
    def find_team(self, team_name):
        for team in self.teams:
            if (same_name(team_name, team.name)):
                return team
        return None

    # Här skrivs de lag som spelas in samt matchens resultat.
    # hjälpfunktionerna kolla stavning och eventuell upprepning
    # samt felkoll på input.
    def change_table(self):
        
        home_team_in = "Hemmalag: "
        away_team_in = "Bortalag: "
        resultIn = "Resultat: "
        result_stripped = ["a", "b"]
        home_team = ""
        away_team = ""

        home_team = self.find_team(get_input(home_team_in)) #hemmalaget skrivs in och felkollas
        home_team = self.check_spelling_dupl(home_team, home_team_in, None)
                   
        away_team = self.find_team(get_input(away_team_in))#bortalaget skrivs in och felkollas
        away_team = self.check_spelling_dupl(away_team, away_team_in, None)
        away_team = self.check_spelling_dupl(away_team, away_team_in, home_team) #här kollas det att det inte är samma bortalag som hemmalag
        
           
        result = get_input(resultIn)# resultat skrivs in
        result_stripped = self.check_result_input(result)# felkoll resultat

        self.get_results(home_team, away_team, result_stripped)  

    #param: team = namn på lag som ska kollas mot ett värde och stavning
    #param: value = värdet (tex None eller redan inskrivet lag) laget kollas mot
    #param: text = det som ska skrivas i ny input förfrågan vid stavfel eller duplicering.
    #return: team = korrekt stavat namn på lag i tabellen
    #Här kollas stavning och eventuell duplicering av inskrivna lag
    def check_spelling_dupl(self, team, text, value):
        while (team == value or team == None):
            print ("Försök igen, kolla upp stavningen på laget.\n(Var även uppmärksam på att det inte kan vara samma bortalag som hemmalag)")
            go_back = get_input("Vill du se tabellen med alla lag, tryck 1. Annars tryck valfri tangent")## om användaren inte vet vilka lag som spelar i serien ser den det här
            if go_back =="1":
                self.print_table()
            team = self.find_team(get_input(text))
        return team
    
    #param: home_team = det först inskrivna laget
    #param: away_team = det andra inskrivna laget
    #param: result = det inskrivna resultatet. som kommer som en lista med två positioner 
    #Lagen får poäng beroende på vilket som gjort flest mål
    #resp om det blivit oavgjort dessutom regleras antal mål gjorda och insläppta samt antal matcher spelade
    def get_results(self, home_team, away_team, result):
        result[0] = int(result[0])
        result[1] = int(result[1])
        result[0] = abs(result[0])# om negativa siffror skrivs in ändras det här till positivt
        result[1] = abs(result[1])
        
        if result[0] > result[1]: # om hemma laget har vunnit
            home_team.points += 3
            home_team.won += 1
            away_team.lost += 1
        elif result[1] > result[0]:#om bortalaget vann
            home_team.lost += 1
            away_team.points += 3
            away_team.won += 1
        else:
            home_team.points += 1 #om det blev oavgjort
            home_team.tie += 1
            away_team.points += 1
            away_team.tie += 1
                 
        home_team.played += 1
        away_team.played += 1
        home_team.goals_won += result[0]
        home_team.goals_lost += result[1]
        away_team.goals_won += result[1]
        away_team.goals_lost += result[0]
            
    
    #returnerar en sorterad lista
    # här ska det sorteras på points, sjunkande ordning. 
    def sort_list(self):
        self.teams = sorted(self.teams, key=attrgetter('points'), reverse=True)
        
        return self.teams

    # felkollar input för resultat inmatningen, splitar på mellanslag eller : och kollar så att listan får 2 positioner
    #param: result = resultatet som användaren skriver in
    #returnerar en lista med 2 p0sitioner
    def check_result_input(self, result):
        if ":" in result:
            result_stripped = result.strip().split(":")
        else:
        #Annars splitta på space
            result_stripped = result.strip().split(" ")

        if len(result_stripped) != 2:
            print("Försök igen!")
            result = get_input("Skriv in resultaten:")
            #Kollar den nya inputen
            return self.check_result_input(result) 
        elif result_stripped[0].isdigit() and result_stripped[1].isdigit():
            return result_stripped
        else:
            #Om vi inte har siffror
            print("Försök igen!")
            result = get_input("Skriv in resultaten:")
            return self.check_result_input(result) #Kollar den nya inputen
        
            

    # en funktion för utskrift av tabellen
    
    def print_table(self):
        self.sort_list()
        print(TAB2 + TAB1 + "S" + TAB1 + "V" + TAB1 + "O" + TAB1 + "F" + TAB1 + " M" + TAB1 + "Po")
        print("\n")
        j = 1
        for i in self.teams:
            print (str(j)+ TAB1 + str(i))
            j += 1
#_____________________Här avslutas klassen_______________________
            
##--------------------HJälpfunktioner----------------------------
TAB1 = "\t" # en tab (tabbar för olika utskrifter)
TAB2 = "\t\t" # två tabbar

## param: text = texten som skrivs ut
## return: info = användarens input 
def get_input(text):
    info = input(text)
    return info

## returnerar true om name1 och name2 är lika            
## tar bort eventuella blanksteg i början eller slutet av strängen och
## gör om alla tecken till små bokstäver
def same_name(name1, name2):
    return name1.strip().lower() == name2.strip().lower()

##return: menu = valen som användaren har
def print_choices():
    menu = "\n" + TAB2 + """     |---------------------------------|
                     | Välj 1 för att mata in Resultat |
                     | Välj 2 för att se tabellen      |
                     | Välj 3 för att spara på fil     |
                     | Välj 4 för att Avsluta          |
                     |---------------------------------|"""
    return menu

## om filen inte finns på datorn skrivs en ny fil här:)
## param: list = fotbollslagslistan.
def create_new_file(list):
    NAME_FILE = 'PremierL.txt'
    with open (NAME_FILE, 'w') as file:
        for team in list:
            file.write(team.name + "," + team.played + "," + team.won + "," + team.tie + "," + team.lost + "," +
                           team.goals_won + "," + team.goals_lost + "," + team.points + "\n")
    
## startfrågor i programmet presenteras, hur vill användaren fortsätta, med en ny säsong eller skriva in  i den redan sparade tabellen    
def initiating_program(football_list):

    NAME_FILE = 'PremierL.txt'
    
    print(TAB2 + """Välkommen till Premier League tabellen!\n\n
        I det här programmet kan du skriva in matchresultatet
        mellan två lag. Resultatet ska skrivas in såhär:
        (Mål hemmalag) 5:3 (Mål bortalag)
        eller med mellanslag mellan lagens totala mål.
        Du kan även se tabellen, spara allt på en fil samt avsluta.\n""")

    start = get_input("\n" + TAB2 + """Tryck 1: Öppna den sparade filen.
                Tryck 2: Börja en ny säsong.
                ----------------------------------------""")
    if start != "2":
        if exists(NAME_FILE)== True: ## här undersöks om filen med ovan namn finns på datorn, om inte fixas en ny fil.
            table = Team_table(NAME_FILE)
        else:
            table = create_new_file(the_list()) ## anropar the_list för lag listan
            table = Team_table(NAME_FILE)
            print("Det fanns tyvärr ingen sparad fil, en ny skapades")
    else:
        table = create_new_file(football_list)
        table = Team_table(NAME_FILE)
    return table

## return: en objektlista med alla fotbollslag skapas och det som ska skrivas i filen vid ny säsong
def the_list():
    football_list = [Team("Chelsea", "0", "0", "0", "0", "0", "0", "0"), Team("Tottenham", "0", "0", "0", "0", "0", "0", "0"),
                     Team("Manchester C", "0", "0", "0", "0", "0", "0", "0"), Team("Liverpool", "0", "0", "0", "0", "0", "0", "0"),
                     Team("Arsenal", "0", "0", "0", "0", "0", "0", "0"),Team("Manchester U", "0", "0", "0", "0", "0", "0", "0"),
                     Team("Everton", "0", "0", "0", "0", "0", "0", "0"),Team("Southampton", "0", "0", "0", "0", "0", "0", "0"),
                     Team("Bournemouth", "0", "0", "0", "0", "0", "0", "0"),Team("WBA", "0", "0", "0", "0", "0", "0", "0"),
                     Team("West Ham", "0", "0", "0", "0", "0", "0", "0"),Team("Leicester", "0", "0", "0", "0", "0", "0", "0"),
                     Team("Stoke", "0", "0", "0", "0", "0", "0", "0"),Team("Crystal Palace", "0", "0", "0", "0", "0", "0", "0"),
                     Team("Swansea", "0", "0", "0", "0", "0", "0", "0"),Team("Burnley", "0", "0", "0", "0", "0", "0", "0"),
                     Team("Watford", "0", "0", "0", "0", "0", "0", "0")]
    return football_list

    
    
       
#--------------här börjar huvudprogrammet-----------       
def main():
    
    NAME_FILE = 'PremierL.txt'


    table = initiating_program(the_list()) ## anropar the_list för lag listan
    choice = get_input(print_choices())
    
    while choice != "4":
        if choice =="1":
            see_table = get_input("\n" + TAB2 + "Vill du se tabellen tryck 1, annars tryck valfri tangent.\n")
            if see_table == "1":
                table.print_table()
            table.change_table()
        elif choice == "2":
            table.print_table()
        elif choice == "3":
            table.save_table(NAME_FILE)
            print("\n" + TAB2 + "Sparat!")
        else:
            print("\n\n" +TAB2 + "Välj endast en siffra mellan 1 och 4!")
        choice = get_input(print_choices())
        
    to_finish = get_input("Vill du spara de senaste ändringarna tryck 1. Tryck valfri tangent för att avsluta programmet utan att spara")
    if (to_finish == "1"):
        table.save_table(NAME_FILE)                      
        print("Ändringarna har sparats och programmet avslutas. Tack för din tid!")
    else:
        print("Programmet avslutas utan att sparas. Tack för din tid!")
       

# huvudprogrammet läggs in i en funktion för att undvika globala variabler      
main()


