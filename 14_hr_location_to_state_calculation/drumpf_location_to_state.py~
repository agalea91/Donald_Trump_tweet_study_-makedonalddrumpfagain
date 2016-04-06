from city_state import city_to_state_dict
import json
import sys
import glob
# finding the state based on geotags
from geopy.geocoders import Nominatim
# the Geonamescache library contains information
# about continents, cities and US states
import geonamescache
# regular expressions for matching tweet
# locations to a dictionary of locations
import re

# binary type output file
f1 = open('find_state_name_drumpf.dat', 'wb')

city_to_state = city_to_state_dict
# remove the cities we don't want
skip = ['Dublin', 'Hamilton', 'Oxford', 'Ontario',
        'Vancouver', 'Ottawa', 'Kingston', 'Jamaica',
        'Columbia', 'Geneva', 'Melbourne', 'Durango',
        'Elgin', 'Moscow', 'Sudbury', 'Holland',
        'Belleville', 'Dublin', 'Amsterdam']
for key in skip:
    if key in city_to_state:
        del city_to_state[key]

# get a dictionary of states: 's'
gc = geonamescache.GeonamesCache()
s = gc.get_us_states()

# link state ID's e.g., TX, AB ...
# to states e.g., Texas, Alabama ...
stateID_to_state = {}
# define a list of states not to
# include as lower case due to chance
# of confusion with other words
no_lower_case_conversion = [
    'HI', 'IN', 'ME', 'MT', 'OH', 'OK', 'OR', 'DE']
for key in s.keys():
    state = s[key]['name']
    stateID_to_state[key] = state
    if key not in no_lower_case_conversion:
        stateID_to_state[key.lower()] = state

# link state name to itself
state_to_state = {}
for key in s.keys():
    state = s[key]['name']
    state_to_state[state] = state

# get a dictionary of cities: 'c'
gc = geonamescache.GeonamesCache()
c = gc.get_cities()

# extract the US city names and coordinates
US_cities = [c[key]['name'] for key in list(c.keys())
          if c[key]['countrycode'] == 'US']
US_longs = [c[key]['longitude'] for key in list(c.keys())
         if c[key]['countrycode'] == 'US']
US_latts = [c[key]['latitude'] for key in list(c.keys())
        if c[key]['countrycode'] == 'US']

def find_word(w):
    return re.compile(r'\b({0})\b'.format(w),
                      flags=re.IGNORECASE)

def find_word_case(w):
    return re.compile(r'\b({0})\b'.format(w))

def find_state_name(f1, tweet_location):
    
    # look for the state ID in
    # the location string
    for key in stateID_to_state.keys():
        # search for the case sensitive state ID
        # to avoid matching things like
        # location = 'here or there' = Oregon
        if find_word_case(key).search(tweet_location):
            f1.write((str(tweet_location) + ' : ' + str(key) + ' -> ' + str(stateID_to_state[key]) + '\n').encode('utf-8'))
            return stateID_to_state[key]
    
    # otherwise look for states
    for key in state_to_state.keys():
        if find_word(key).search(tweet_location):
            f1.write((str(tweet_location) + ' : ' + str(key) + ' -> ' + str(state_to_state[key]) + '\n').encode('utf-8'))
            return state_to_state[key]
        
    # otherwise look for cities
    for key in city_to_state.keys():
        if find_word(key).search(tweet_location):
            f1.write((str(tweet_location) + ' : ' + str(key) + ' -> ' + str(city_to_state[key]) + '\n').encode('utf-8'))
            return city_to_state[key]
    
    # otherwise return empty string
    return ''


f3 = open('drumpf_location_to_state.dat', 'w')
file_root = '../drumpf_tweets/#make'
print(list(glob.iglob(file_root+'*')))
tweet_files = list(glob.iglob(file_root+'*'))
for file in tweet_files:
    print('Starting file:', file)
    with open(file, 'r') as f2:
        lines = f2.readlines()
        for line in lines:
            loc = json.loads(line)['user']['location']
            state = find_state_name(f1, loc)
            f3.write(state+'\n')
    print('Done file:', file)


f1.close()
f3.close()
