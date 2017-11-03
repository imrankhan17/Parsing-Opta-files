

```python
import csv
import xml.etree.ElementTree as et
import numpy as np
from datetime import datetime as dt
```


```python
tree = et.ElementTree(file = "Bolton_ManCityF24.xml")
games = tree.getroot()
```


```python
match_details = games[0].attrib
match_details
```




    {'away_team_id': '43',
     'away_team_name': 'Manchester City',
     'competition_id': '8',
     'competition_name': 'English Barclays Premier League',
     'game_date': '2011-08-21T16:00:00',
     'home_team_id': '30',
     'home_team_name': 'Bolton Wanderers',
     'id': '360481',
     'matchday': '2',
     'period_1_start': '2011-08-21T16:00:38',
     'period_2_start': '2011-08-21T17:03:47',
     'season_id': '2011',
     'season_name': 'Season 2011/2012'}




```python
tree2 = et.ElementTree(file = "Bolton_ManCityF7.xml")
soccerfeed = tree2.getroot()
```

List of players


```python
player_ids = []
player_names = []

for child in soccerfeed:
    for grchild in child:
        if grchild.tag == "Team":
            for grgrchild in grchild:
                if grgrchild.tag == "Player":
                    player_ids.append(grgrchild.attrib["uID"].lstrip('p'))
                                    
                    for grgrgrchild in grgrchild:
                        player_names.append(grgrgrchild[0].text + " " + grgrgrchild[-1].text)
                        
player_dict = dict(zip(player_ids, player_names))
player_dict
```




    {'10089': 'Martin Petrov',
     '105088': 'Adam Blakeman',
     '1344': u'Jussi J\xe4\xe4skel\xe4inen',
     '14664': u'Gnegneri Yaya Tour\xe9',
     '14668': 'Nigel Reo-Coker',
     '15157': 'James Milner',
     '15188': 'Darren Pratley',
     '15749': 'Joe Hart',
     '1587': 'Zat Knight',
     '1615': 'Robbie Blake',
     '1632': 'Gareth Barry',
     '17336': u'Ga\xebl Clichy',
     '17476': 'Vincent Kompany',
     '18428': 'Chris Eagles',
     '19419': 'Gary Cahill',
     '19930': 'Mark Davies',
     '19958': 'David Wheater',
     '19959': 'Adam Johnson',
     '2004': 'Paul Robinson',
     '20312': u'Carlos T\xe9vez',
     '20492': 'Micah Richards',
     '20658': 'Pablo Zabaleta',
     '20664': 'David Silva',
     '27696': 'Fabrice Muamba',
     '28183': u'Gr\xe9tar Steinsson',
     '3630': 'Kevin Davies',
     '37572': u'Sergio Ag\xfcero',
     '42493': 'Mario Balotelli',
     '42544': 'Edin Dzeko',
     '42593': 'Aleksandar Kolarov',
     '45175': 'Adam Bogdan',
     '56827': 'Costel Pantilimon',
     '65807': 'Stefan Savic',
     '7551': 'Joleon Lescott',
     '82263': 'Marcos Alonso',
     '9765': 'Ivan Klasnic'}



Match preview summary


```python
print "%s v %s, %s %s" % (match_details["home_team_name"],
                          match_details["away_team_name"],
                          match_details["competition_name"][8:],
                          match_details["season_name"][7:])

print "Venue: %s" % venue

print "Date: %s" % dt.strftime(dt.strptime(match_details["game_date"], '%Y-%m-%dT%H:%M:%S'),
                               "%A %d %B %Y")

print "Kick-off: %s" % dt.strftime(dt.strptime(match_details["game_date"], '%Y-%m-%dT%H:%M:%S'),
                               "%I%p").lstrip("0")
```

    Bolton Wanderers v Manchester City, Barclays Premier League 2011/2012
    Venue: Reebok Stadium
    Date: Sunday 21 August 2011
    Kick-off: 4PM



```python
team_dict = {match_details["home_team_id"]: match_details["home_team_name"],
             match_details["away_team_id"]: match_details["away_team_name"]}

print team_dict
```

    {'30': 'Bolton Wanderers', '43': 'Manchester City'}



```python
# PASSES

passes_x = []
passes_y = []
passes_outcome = []
passes_min = []
passes_sec = []
passes_period = []
passes_team = []
passes_x_end = []
passes_y_end = []
passes_length = []
passes_angle = []
passes_zone = []
pass_real = []
pass_player = []

for game in games:
    for event in game:
        
        if event.attrib.get("type_id") == '1':
            
            passes_x.append(event.attrib.get("x"))
            passes_y.append(event.attrib.get("y"))
            passes_outcome.append(event.attrib.get("outcome"))
            passes_min.append(event.attrib.get("min"))
            passes_sec.append(event.attrib.get("sec"))
            passes_period.append(event.attrib.get("period_id"))
            passes_team.append(team_dict[event.attrib.get("team_id")])
            pass_player.append(player_dict[event.attrib.get("player_id")].encode('utf-8'))
            
            for q in event:
                
                qualifier = q.attrib.get("qualifier_id")
                
                if qualifier == "140":
                    passes_x_end.append(q.attrib.get("value"))
                if qualifier == "141":
                    passes_y_end.append(q.attrib.get("value"))
                if qualifier == "212":
                    passes_length.append(q.attrib.get("value"))
                if qualifier == "213":
                    passes_angle.append(q.attrib.get("value"))
                if qualifier == "56":
                    passes_zone.append(q.attrib.get("value"))
                    
                             
passes_df = np.array(zip(passes_team, pass_player, passes_period, passes_min, passes_sec, passes_zone, passes_x, 
                         passes_y, passes_x_end, passes_y_end, passes_length, passes_angle, 
                         passes_outcome))
print passes_df

fieldnames = ["team", "player", "period", "min", "sec", "pass zone", "x", "y", "x_end", "y_end",
              "pass length", "pass angle", "outcome"]

with open("pass_data_%s_%s.csv" % (match_details["home_team_name"], match_details["away_team_name"]), 
          "wb") as passes_csv:
        csv_file = csv.writer(passes_csv)
        csv_file.writerow(fieldnames)
        for i in range(len(passes_df)):
            csv_file.writerow(passes_df[i])
```

    [['Manchester City' 'Sergio Ag\xc3\xbcero' '1' ..., '2.5' '6.0' '1']
     ['Manchester City' 'David Silva' '1' ..., '27.5' '2.4' '1']
     ['Manchester City' 'Aleksandar Kolarov' '1' ..., '23.2' '6.1' '0']
     ..., 
     ['Manchester City' 'Micah Richards' '2' ..., '31.0' '0.3' '1']
     ['Manchester City' 'Gnegneri Yaya Tour\xc3\xa9' '2' ..., '5.6' '4.6' '1']
     ['Bolton Wanderers' 'Jussi J\xc3\xa4\xc3\xa4skel\xc3\xa4inen' '2' ...,
      '66.4' '6.2' '1']]



```python
# GOALS

goal_x = []
goal_y = []
goal_zone = []
goal_outcome = []
goal_min = []
goal_sec = []
goal_period = []
goal_team = []
goalmouth_y = []
goalmouth_z = []
goal_assisted = []
body_part = []
goal_player = []

body_dict = {"15": "head",
            "72": "left foot",
            "20": "right foot",
            "21": "other body part"}

for game in games:
    for event in game:
        
        if event.attrib.get("type_id") == '16':
            
            goal_x.append(event.attrib.get("x"))
            goal_y.append(event.attrib.get("y"))
            goal_outcome.append(event.attrib.get("outcome"))
            goal_min.append(event.attrib.get("min"))
            goal_sec.append(event.attrib.get("sec"))
            goal_period.append(event.attrib.get("period_id"))
            goal_team.append(team_dict[event.attrib.get("team_id")])
            goal_player.append(player_dict[event.attrib.get("player_id")].encode('utf-8'))
            
            for q in event:
                
                qualifier = q.attrib.get("qualifier_id")
                
                
                if qualifier == "103":
                    goalmouth_z.append(q.attrib.get("value"))
                if qualifier == "102":
                    goalmouth_y.append(q.attrib.get("value"))
                if qualifier == "56":
                    goal_zone.append(q.attrib.get("value"))
                if qualifier in ["15", "72", "20", "21"]:
                    body_part.append(body_dict[qualifier])
                
                
                             
goal_df = np.array(zip(goal_team, goal_player, goal_period, goal_min, goal_sec, body_part, goal_zone, goal_x, 
                         goal_y, goalmouth_y, goalmouth_z, goal_outcome))
print goal_df

goal_fieldnames = ["team", "player", "period", "min", "sec", "body part", "zone", "x", "y", 
                   "goalmouth y", "goalmouth z", "outcome", "assisted"]

with open("goal_data_%s_%s.csv" % (match_details["home_team_name"], match_details["away_team_name"]), 
          "wb") as goal_csv:
        csv_file = csv.writer(goal_csv)
        csv_file.writerow(goal_fieldnames)
        for i in range(len(goal_df)):
            csv_file.writerow(goal_df[i])
```

    [['Manchester City' 'David Silva' '1' '25' '14' 'left foot' 'Center' '78.4'
      '38.2' '52.1' '22.2' '1']
     ['Manchester City' 'Gareth Barry' '1' '36' '59' 'left foot' 'Center'
      '74.8' '46.4' '46.4' '30.6' '1']
     ['Bolton Wanderers' 'Ivan Klasnic' '1' '38' '54' 'left foot' 'Center'
      '91.4' '57.3' '50.3' '2.8' '1']
     ['Manchester City' 'Edin Dzeko' '2' '46' '39' 'right foot' 'Center' '90.6'
      '34.7' '51.9' '6.9' '1']
     ['Bolton Wanderers' 'Kevin Davies' '2' '62' '6' 'head' 'Center' '90.7'
      '48.8' '46.0' '9.7' '1']]



```python
# ALL SHOTS

shot_name = []
shot_x = []
shot_y = []
shot_zone = []
shot_min = []
shot_sec = []
shot_period = []
shot_team = []
goalmouth_y = []
goalmouth_z = []
saved_x = []
saved_y = []
body_part = []
shot_play = []
shot_player = []

shot_dict = {'13': 'Shot off target',
             '14': 'Post',
             '15': 'Shot saved',
             '16': 'Goal'}

body_dict = {"15": "head",
            "72": "left foot",
            "20": "right foot",
            "21": "other body part"}

shot_play_dict = {'22': 'regular play',
            '23': 'fast break',
            '24': 'set piece',
            '25': 'from corner',
            '26': 'free kick',
            '96': 'corner situation',
            '112': 'scramble',
            '160': 'throw-in set piece',
            '9': 'penalty',
            '28': 'own goal'}

for game in games:
    
    for event in game:
        
        if event.attrib.get("type_id") in ['13', '14', '16']:
                    
            shot_name.append(shot_dict[event.attrib.get("type_id")])
            shot_x.append(event.attrib.get("x"))
            shot_y.append(event.attrib.get("y"))
            shot_min.append(event.attrib.get("min"))
            shot_sec.append(event.attrib.get("sec"))
            shot_period.append(event.attrib.get("period_id"))
            shot_team.append(team_dict[event.attrib.get("team_id")])
            shot_player.append(player_dict[event.attrib.get("player_id")].encode('utf-8'))
            
            for q in event:
                
                qualifier = q.attrib.get("qualifier_id")
                if qualifier == '102':
                    saved_x.append('')
                    saved_y.append('')
                    goalmouth_y.append(q.attrib.get("value"))
                if qualifier == '103':
                    goalmouth_z.append(q.attrib.get("value"))
                if qualifier in body_dict.keys():
                    body_part.append(body_dict[qualifier])
                if qualifier in shot_play_dict.keys():
                    shot_play.append(shot_play_dict[qualifier])
                                   
        if event.attrib.get("type_id") == '15':
                    
            shot_name.append(shot_dict[event.attrib.get("type_id")])
            shot_x.append(event.attrib.get("x"))
            shot_y.append(event.attrib.get("y"))
            shot_min.append(event.attrib.get("min"))
            shot_sec.append(event.attrib.get("sec"))
            shot_period.append(event.attrib.get("period_id"))
            shot_team.append(team_dict[event.attrib.get("team_id")])
            shot_player.append(player_dict[event.attrib.get("player_id")].encode('utf-8'))
                        
            
            for q in event:
                
                qualifier = q.attrib.get("qualifier_id")
                if qualifier == '146':
                    goalmouth_y.append('')
                    goalmouth_z.append('')
                    saved_x.append(q.attrib.get("value"))
                if qualifier == '147':
                    saved_y.append(q.attrib.get("value"))
                if qualifier in ["15", "72", "20", "21"]:
                    body_part.append(body_dict[qualifier])
                if qualifier in shot_play_dict.keys():
                    shot_play.append(shot_play_dict[qualifier])
                               
                             
shot_df = np.array(zip(shot_team, shot_player, shot_period, shot_min, shot_sec, shot_play, shot_name, body_part, shot_x, shot_y, 
                       goalmouth_y, goalmouth_z, saved_x, saved_y))
    
print shot_df

shot_fieldnames = ["team", "player", "period", "min", "sec", "shot play", "shot type", "body part", "x", "y", "goalmouth y", 
                   "goalmouth z", "saved x", "saved y"]

with open("shot_data_%s_%s.csv" % (match_details["home_team_name"], match_details["away_team_name"]), 
          "wb") as shot_csv:
        csv_file = csv.writer(shot_csv)
        csv_file.writerow(shot_fieldnames)
        for i in range(len(shot_df)):
            csv_file.writerow(shot_df[i])
```

    [['Bolton Wanderers' 'Chris Eagles' '1' '2' '59' 'free kick' 'Shot saved'
      'right foot' '75.9' '66.6' '' '' '98.2' '50.2']
     ['Bolton Wanderers' 'Zat Knight' '1' '10' '3' 'from corner'
      'Shot off target' 'right foot' '87.3' '41.4' '41.1' '9.7' '' '']
     ['Bolton Wanderers' 'Kevin Davies' '1' '10' '39' 'regular play'
      'Shot saved' 'head' '93.0' '54.8' '' '' '95.2' '54.0']
     ['Manchester City' 'James Milner' '1' '14' '39' 'regular play'
      'Shot saved' 'right foot' '96.1' '52.5' '' '' '97.6' '52.0']
     ['Manchester City' 'David Silva' '1' '16' '26' 'regular play' 'Shot saved'
      'left foot' '81.2' '27.6' '' '' '98.1' '47.3']
     ['Manchester City' 'Sergio Ag\xc3\xbcero' '1' '17' '52' 'regular play'
      'Shot off target' 'right foot' '92.7' '56.4' '50.7' '70.8' '' '']
     ['Bolton Wanderers' 'Chris Eagles' '1' '21' '14' 'set piece' 'Shot saved'
      'right foot' '79.0' '30.1' '' '' '80.2' '31.9']
     ['Manchester City' 'Sergio Ag\xc3\xbcero' '1' '24' '43' 'regular play'
      'Shot saved' 'right foot' '89.8' '24.3' '' '' '90.7' '27.1']
     ['Manchester City' 'David Silva' '1' '25' '14' 'throw-in set piece' 'Goal'
      'left foot' '78.4' '38.2' '52.1' '22.2' '' '']
     ['Manchester City' 'Sergio Ag\xc3\xbcero' '1' '34' '56' 'regular play'
      'Shot off target' 'head' '92.8' '48.0' '57.8' '8.3' '' '']
     ['Manchester City' 'Gareth Barry' '1' '36' '59' 'from corner' 'Goal'
      'left foot' '74.8' '46.4' '46.4' '30.6' '' '']
     ['Bolton Wanderers' 'Ivan Klasnic' '1' '38' '54' 'regular play' 'Goal'
      'left foot' '91.4' '57.3' '50.3' '2.8' '' '']
     ['Manchester City' 'Edin Dzeko' '2' '46' '39' 'regular play' 'Goal'
      'right foot' '90.6' '34.7' '51.9' '6.9' '' '']
     ['Manchester City' 'James Milner' '2' '49' '18' 'regular play'
      'Shot saved' 'right foot' '79.3' '65.9' '' '' '98.1' '51.1']
     ['Bolton Wanderers' 'Chris Eagles' '2' '51' '11' 'regular play'
      'Shot off target' 'right foot' '88.4' '70.6' '58.0' '5.6' '' '']
     ['Manchester City' 'James Milner' '2' '52' '36' 'regular play'
      'Shot off target' 'right foot' '76.5' '45.3' '39.8' '37.5' '' '']
     ['Manchester City' 'Edin Dzeko' '2' '58' '39' 'from corner'
      'Shot off target' 'head' '92.3' '46.4' '37.5' '6.9' '' '']
     ['Manchester City' 'Aleksandar Kolarov' '2' '60' '31' 'from corner'
      'Shot saved' 'left foot' '75.1' '74.0' '' '' '88.9' '58.1']
     ['Bolton Wanderers' 'Kevin Davies' '2' '62' '6' 'set piece' 'Goal' 'head'
      '90.7' '48.8' '46.0' '9.7' '' '']
     ['Manchester City' 'Gnegneri Yaya Tour\xc3\xa9' '2' '67' '7'
      'regular play' 'Shot off target' 'right foot' '71.4' '53.9' '34.4' '58.3'
      '' '']
     ['Manchester City' 'David Silva' '2' '68' '41' 'regular play' 'Shot saved'
      'left foot' '92.7' '56.7' '' '' '94.9' '55.6']
     ['Manchester City' 'Aleksandar Kolarov' '2' '75' '6' 'regular play'
      'Shot off target' 'left foot' '85.2' '65.1' '32.5' '1.4' '' '']
     ['Manchester City' 'Adam Johnson' '2' '83' '9' 'regular play'
      'Shot off target' 'right foot' '95.3' '35.5' '59.9' '1.4' '' '']
     ['Manchester City' 'Carlos T\xc3\xa9vez' '2' '85' '41' 'regular play'
      'Shot off target' 'left foot' '87.5' '55.9' '0.2' '1.4' '' '']
     ['Manchester City' 'Carlos T\xc3\xa9vez' '2' '91' '53' 'regular play'
      'Shot saved' 'left foot' '91.8' '77.2' '' '' '99.7' '56.0']]

