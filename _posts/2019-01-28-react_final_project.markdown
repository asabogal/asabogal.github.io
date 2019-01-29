---
layout: post
title:      "React Final Project"
date:       2019-01-29 04:07:47 +0000
permalink:  react_final_project
---


So far I’ve built a Ruby CLI application, two traditional CRUD applications; one on Sinatra and a second on Rails. My previous project utilized a backend API to fetch data from and external API and render that data to the user utilizing traditional vanilla Javascript and JQuery. All these applications were designed with typical web-browser as the interface, and so for my last project I thought it would be good practice to build an app with an interface that would be user friendly in both the browser and mobile devices. React’s component rendering approach was the perfect way to achieve this. 

I decided to build a workout Timer application based on the High-Intensity-Interval-Training approach. This is when a person does a high intensity cardio exercise for a certain time and it’s immediately followed by a recovery period. This routine is then repeated a number of times or until failure. This app allows the user to set this intervals and the number of repetitions, and keep track of the workout. Pre-sets are given or the user can customize his/her own settings.

I knew I would be building a single page application for this timer, so I initially sketched the main page and divided the sections into potential React containers and components. I figured the main container would be a Timer container that could be divided into three components: the Timer Display, Timer Settings, and Timer Controllers. And so I began coding the main Timer logic inside the Timer container.

I thought it would be a good idea to build the Timer code as a React application and then once I had it all working, I would refactor the code and to incorporate Redux. I soon realized this was not the best approach. Redux’s store should be managed by actions and dispatch, and because the Timer logic was initially written on traditional Javascript functions, and because it relied heavily on the TImer’s local state i.e. every time a Timer digit changes or the settings are changed or saved, almost all the logic had to be re-coded into actions and reducer logic. This took a while to redo and maintain the functional logic but it was a good coding experience and practice.

I decided to setup two Reducers via Redux’s combineReducers helper function. One reducer will manage the global state or timer settings responsible for the timer logic and display, and another to write and hold the data from the API backend via async actions. I used ‘react-thunk’ middleware library to handle async actions and dispatch, and to keep track of the Redux store I used ‘redux-logger’ —a middleware that console logs every dispatched action and the before and after state of the redux store. The index.js file looks like this:


```
import React from 'react';
import ReactDOM from 'react-dom';
import Routes from './Routes';
import * as serviceWorker from './serviceWorker';
import 'bootstrap/dist/css/bootstrap.css';
import './index.css';

import rootReducer from './reducers/rootReducer.js'
import { Provider } from 'react-redux'
import { applyMiddleware, createStore } from 'redux'
import logger from 'redux-logger'
import thunk from 'redux-thunk'

const store = createStore(rootReducer, applyMiddleware(thunk))


ReactDOM.render(
  <Provider store={store}>
     <Routes />
  </Provider>
 , 
  document.getElementById('root'));

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: http://bit.ly/CRA-PWA
serviceWorker.unregister();
```

For styling I used ‘reactstrap’ a framework built around the popular Bootsrap front-end framework. This is very simple to setup and provides design examples on the documentation: https://reactstrap.github.io
It uses state-less components to style your React app. One design feature I was sure I wanted to implement and hadn’t used in previous projects, was the Modal pop-up. Here’s an example of a Modal containing the customize settings form:

Modal Form:

```
import React from 'react';
import { Button, Modal, ModalHeader, ModalBody } from 'reactstrap';
import CustomizeForm from '../components/Timer/CustomizeForm'

class ModalForm extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      modal: false
    };
  }

  toggle = () => {
    this.setState({
      modal: !this.state.modal
    });
  }

  render() {
    return (
      <div>
        <Button outline color="info" size="sm" onClick={this.toggle}>Customize</Button>
        <Modal  isOpen={this.state.modal} toggle={this.toggle} >
          <ModalHeader  toggle={this.toggle}>Customize Settings:</ModalHeader>
          <ModalBody >
            <CustomizeForm setCustomSettings={this.props.setCustomSettings} toggle={this.toggle} />
          </ModalBody>
        </Modal>
      </div>
    );
  }
}

export default ModalForm;
```

Customize Settings Form:

```
import React from 'react';
import { Form, FormGroup, Input, Label } from 'reactstrap';
import { createWorkout } from '../../actions/apiActions'

import { connect } from 'react-redux'

class CustomizeForm extends React.Component {
  constructor(props) {
    super(props);

    this.state = { 
      name: '',
      sets: 0,
      running_sets: 0,
      interval: 0,
      rest: 0,
      running: false,
      running_time: 0
      };
  }

  handleChange = (event) => {
    const { name, value } = event.target
    this.setState({
      [name]: value
    })
  }

  handleSubmit = (event) => {
    event.preventDefault()
    this.props.createWorkout(this.state)
    this.props.setCustomSettings(this.state)
    this.props.toggle()
  }

  render() {
    return (
      <div>
          <Form onSubmit={this.handleSubmit}>
            <FormGroup>
              <Label >Workout Name</Label>
              <Input onChange={this.handleChange} 
              name="name"
              value={this.state.name}
              bsSize="sm"
              required />
            </FormGroup>
            <FormGroup>
              <Label >Number of Sets</Label>
              <Input onChange={this.handleChange} 
              type="number"
              min="1"
              name="sets"
              value={this.state.sets}
              bsSize="sm" />
            </FormGroup>
            <FormGroup>
              <Label >Interval Length (secs)</Label>
              <Input onChange={this.handleChange}
              type="number"
              min="3"
              max="60" 
              name="interval"
              value={this.state.interval}
              bsSize="sm" />
            </FormGroup>
            <FormGroup>
              <Label >Rest Length (secs)</Label>
              <Input onChange={this.handleChange}
              type="number"
              min="3"
              max="60" 
              name="rest"
              value={this.state.rest}
              bsSize="sm" />
            </FormGroup>
            <FormGroup>
              
              <Input type="submit" value="Update Settings" bsSize="sm"></Input>
            </FormGroup>
        </Form>
      </div>
    );
  }
}

export default connect(null, { createWorkout })(CustomizeForm);
```

To setup my Routes, I built a “Routes” container that will be the main App container rendered on the DOM root div. Depending on the browser address, the app will render its corresponding component:

```
import React from 'react'
import { BrowserRouter, Route, Switch } from 'react-router-dom'
import App from './containers/App'
import Timer from './containers/Timer'
import CustomizeForm from './components/Timer/CustomizeForm'
import ModalAbout from './containers/ModalAbout'

const AppRoutes = () => {

 return (
   <BrowserRouter>
    <Switch>
      <Route exact path= '/' component={App}/>
      <Route path= '/timer' component={Timer}/>
      <Route path= '/workouts/new' component={CustomizeForm}/>
      <Route path= '/about' component={ModalAbout}/>
    </Switch>
   </BrowserRouter>
 )
}

export default AppRoutes;
```

Overall I am very happy with the end result. I achieved exactly what I wanted to achieve. The App looks like my initial sketch. 
Eventually I will add additional functionality like a user profile and further customization, like timer color and alert sounds, but for now its functionality meets the project requirements.
