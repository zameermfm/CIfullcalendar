# CIfullcalendar
FullCalendar Jquery Calendar plugin integration for PHP Codeigniter framework

<b>Copy Event.php to Application\Library folder</b>

If you are loading your Calendar data from a database this example will explain on how to integrate,
In the following example, we assume there is a controller named <i>SampleController</I> and the controller has a function named <i>viewcalendar</i> which load view <i>fullcalendarexample</i> (the view where full calendar is integrated), also a model named <i>samplemodel</i> and the model has a function <i>getCalendarEvents()</i> to get all calendar events in normal <i>datetime</i> mysql format. 
	
	//Controller
	class SampleController extends CI_Controller {

	function __construct()
	{
			parent::__construct();
			$this->load->model('samplemodel');
			$this->load->library('event'); //Loading the event library given on the repository
	}
	function viewcalendar()
	{
	    $this->load->view('fullcalendarexample');
	}
	

add this AJAX Function in this controller

	public function get_calendar_data(){
		if ($this->input->is_ajax_request()) 
			{
				$timezone = null; //or you can load from somewhere
				$calendardata = $this->samplemodel->getCalendarEvents();
		
				$json = array();
        			$mydata = $this->input->post('mydata'); //recieving the ajax post data
				$format = 'DATE_ISO8601';

				foreach ($calendardata as $key =>$value) {
					$title = $calendardata[$key]->title;
					$startTime = human_to_unix($calendardata[$key]->event_start);
					$eventStart = standard_date($format, $startTime);
					$endTime = human_to_unix($calendardata[$key]->event_end);
					$eventEnd = standard_date($format, $endTime);
        				$url = "http://www.google.com";
        				
					$json[] = array(
				        'title' => $title,
				        'start' =>$eventStart,
				        'end' => $eventEnd,
				        'url' => $url

				    );

				}
				$results = array();
				foreach ($json as $array) {

					// Convert the input array into a useful Event object
					$event = $this->event->init($array, $timezone);

					// If the event is in-bounds, add it to the output
						$results[] = $this->event->toArray($event);
				}

				echo json_encode($results,JSON_UNESCAPED_SLASHES);
			}
		else 
			{
			    redirect('error'); //there should be a controller called error
			}

	}


in the view where FullCalendar is integrated you can add these codes to the Jquery script to get to the Ajax function and
recieve calendar data.


	function renderCalendar(sentdata) {
		
			$('#calendar').fullCalendar({
				header: {
					left: 'prev,next today',
					center: 'title',
					right: 'month,agendaWeek,agendaDay'
				},

				
				eventLimit: true, // allow "more" link when too many events
				events: {
					type: 'POST',
					data: {
		                mydata: sentdata // if you want to send data, it is recieved back at the controller
		            },
					url: "<?php echo base_url()."samplecontroller/get_calendar_events";?>"
				},
				


			});
		}

		renderCalendar();
