import React, { useEffect, useState } from 'react';
import { StyleSheet, Text, View, TextInput, Button, ActivityIndicator, ScrollView, ImageBackground } from 'react-native';
import { parseString } from 'react-native-xml2js'; 

const App = () => {
  const [weatherData, setWeatherData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [query, setQuery] = useState('London');
  const [search, setSearch] = useState('London');

  const fetchWeather = (location) => {
    setLoading(true);
    const apiKey = '783a00d3566247b789c125422243006 ';  
    const url = `http://api.weatherapi.com/v1/current.xml?key=${apiKey}&q=${location}`;

    fetch(url)
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        return response.text();
      })
      .then(text => {
        console.log('Response text:', text);
        parseString(text, { explicitArray: false }, (err, result) => {
          if (err) {
            console.error('XML parse error:', err);
            setError(err.message);
          } else {
            console.log('Parsed data:', result);
            setWeatherData(result.root);
            setError(null);
          }
          setLoading(false);
        });
      })
      .catch((error) => {
        console.error('Fetch error:', error);
        setError(error.message);
        setLoading(false);
      });
  };

  useEffect(() => {
    fetchWeather(query);
  }, [query]);

  const handleSearch = () => {
    setQuery(search);
  };

  const getBackgroundImage = () => {
    if (!weatherData) return require('./assets/default.jpg');
    const condition = weatherData.current.condition.text.toLowerCase();
    if (condition.includes('rain')) return require('./assets/rain.jpeg');
    if (condition.includes('cloud')) return require('./assets/cloudy1.jpg');
    if (condition.includes('sun')) return require('./assets/sunny.jpg');
    return require('./assets/default.jpg');
  };

  return (
    <ImageBackground source={getBackgroundImage()} style={styles.background} resizeMode="cover">
      <ScrollView contentContainerStyle={styles.container}>
        <TextInput
          style={styles.input}
          placeholder="Enter location"
          value={search}
          onChangeText={setSearch}
        />
        <Button title="Search" onPress={handleSearch} />
        {loading ? (
          <ActivityIndicator size="large" color="#0000ff" />
        ) : error ? (
          <Text style={styles.errorText}>Error fetching data: {error}</Text>
        ) : weatherData ? (
          <>
            <Text style={styles.title}>Weather in {weatherData.location.name}</Text>
            <Text style={styles.info}>Temperature: {weatherData.current.temp_c}°C</Text>
            <Text style={styles.info}>Condition: {weatherData.current.condition.text}</Text>
            <Text style={styles.info}>Wind Speed: {weatherData.current.wind_kph} kph</Text>
          </>
        ) : null}
      </ScrollView>
    </ImageBackground>
  );
};

export default App;

const styles = StyleSheet.create({
  background: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    width: '100%',
    height: '100%',
  },
  container: {
    flexGrow: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: 'rgba(255, 255, 255, 0.8)',
    padding: 20,
    borderRadius: 10,
    margin: 20,
  },
  input: {
    height: 40,
    borderColor: 'gray',
    borderWidth: 1,
    paddingLeft: 8,
    marginBottom: 20,
    width: '80%',
    borderRadius: 5,
    backgroundColor: 'white',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  info: {
    fontSize: 18,
    marginBottom: 10,
    textAlign: 'center',
  },
  errorText: {
    fontSize: 18,
    color: 'red',
    textAlign: 'center',
  },
});
