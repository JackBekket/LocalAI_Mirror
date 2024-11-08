# core/http/elements/buttons.go  
## Package: elements  
  
### Imports:  
  
- strings  
- github.com/chasefleming/elem-go  
- github.com/chasefleming/elem-go/attrs  
- github.com/mudler/LocalAI/core/gallery  
  
### External Data, Input Sources:  
  
- galleryName (string)  
- m (gallery.GalleryModel)  
- galleryID (string)  
  
### Summary:  
  
This package provides functions to create buttons for various actions related to models in a gallery.  
  
#### installButton:  
  
This function creates an install button for a given gallery name. The button has a specific set of attributes, including data attributes for ripple effects, class for styling, and a data attribute for the target element to be swapped. The button also has a post action that sends the gallery name to the specified URL.  
  
#### reInstallButton:  
  
This function creates a reinstall button for a given gallery name. Similar to the install button, it has a set of attributes for styling and functionality. The button also has a post action that sends the gallery name to the specified URL.  
  
#### infoButton:  
  
This function creates an info button for a given gallery model. The button has a set of attributes for styling and functionality, including a data attribute for the modal target and toggle. The button also has an icon and text for the info action.  
  
#### deleteButton:  
  
This function creates a delete button for a given gallery ID. The button has a set of attributes for styling and functionality, including a confirmation message for the delete action. The button also has a post action that sends the gallery ID to the specified URL.  
  
#### dropBadChars:  
  
This function replaces any "@" characters in a string with "__" to avoid issues with Javascript/HTMX.  
  
  
  
# core/http/elements/gallery.go  
```  
package elements  
  
import (  
	"fmt"  
  
	"github.com/chasefleming/elem-go"  
	"github.com/chasefleming/elem-go/attrs"  
	"github.com/mudler/LocalAI/core/gallery"  
	"github.com/mudler/LocalAI/core/services"  
)  
  
const (  
	noImage = "https://upload.wikimedia.org/wikipedia/commons/6/65/No-Image-Placeholder.svg"  
)  
  
func cardSpan(text, icon string) elem.Node {  
	return elem.Span(  
		attrs.Props{  
			"class": "inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2 mb-2",  
		},  
		elem.I(attrs.Props{  
			"class": icon + " pr-2",  
		}),  
  
		elem.Text(bluemonday.StrictPolicy().Sanitize(text)),  
	)  
}  
  
func searchableElement(text, icon string) elem.Node {  
	return elem.Form(  
		attrs.Props{},  
		elem.Input(  
			attrs.Props{  
				"type":  "hidden",  
				"name":  "search",  
				"value": text,  
			},  
		),  
		elem.Span(  
			attrs.Props{  
				"class": "inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2 mb-2 hover:bg-gray-300 hover:shadow-gray-2",  
			},  
			elem.A(  
				attrs.Props{  
					//	"name":      "search",  
					//	"value":     text,  
					//"class":     "inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2 mb-2",  
					"href":      "#!",  
					"hx-post":   "/browse/search/models",  
					"hx-target": "#search-results",  
					// TODO: this doesn't work  
					//	"hx-vals":      `{ \"search\": \"` + text + `\" }`,  
					"hx-indicator": ".htmx-indicator",  
				},  
				elem.I(attrs.Props{  
					"class": icon + " pr-2",  
				}),  
				elem.Text(bluemonday.StrictPolicy().Sanitize(text)),  
			),  
		),  
	)  
}  
  
func link(text, url string) elem.Node {  
	return elem.A(  
		attrs.Props{  
			"class":  "text-base leading-relaxed text-gray-500 dark:text-gray-400",  
			"href":   url,  
			"target": "_blank",  
		},  
		elem.I(attrs.Props{  
			"class": "fas fa-link pr-2",  
		}),  
		elem.Text(bluemonday.StrictPolicy().Sanitize(text)),  
	)  
}  
  
type ProcessTracker interface {  
	Exists(string) bool  
	Get(string) string  
}  
  
func modalName(m *gallery.GalleryModel) string {  
	return m.Name + "-modal"  
}  
  
func modelDescription(m *gallery.GalleryModel) elem.Node {  
	urls := []elem.Node{}  
	for _, url := range m.URLs {  
		urls = append(urls,  
			elem.Li(attrs.Props{}, link(url, url)),  
		)  
	}  
  
	tagsNodes := []elem.Node{}  
	for _, tag := range m.Tags {  
		tagsNodes = append(tagsNodes,  
			searchableElement(tag, "fas fa-tag"),  
		)  
	}  
  
	return elem.Div(  
		attrs.Props{  
			"class": "p-6 text-surface dark:text-white",  
		},  
		elem.H5(  
			attrs.Props{  
				"class": "mb-2 text-xl font-bold leading-tight",  
			},  
			elem.Text(bluemonday.StrictPolicy().Sanitize(m.Name)),  
		),  
		elem.Div( // small description  
			attrs.Props{  
				"class": "mb-4 text-sm truncate text-base",  
			},  
			elem.Text(bluemonday.StrictPolicy().Sanitize(m.Description)),  
		),  
  
		elem.Div(  
			attrs.Props{  
				"id":          modalName(m),  
				"tabindex":    "-1",  
				"aria-hidden": "true",  
				"class":       "hidden overflow-y-auto overflow-x-hidden fixed top-0 right-0 left-0 z-50 justify-center items-center w-full md:inset-0 h-[calc(100%-1rem)] max-h-full",  
			},  
			elem.Div(  
				attrs.Props{  
					"class": "relative p-4 w-full max-w-2xl max-h-full",  
				},  
				elem.Div(  
					attrs.Props{  
						"class": "relative p-4 w-full max-w-2xl max-h-full bg-white rounded-lg shadow dark:bg-gray-700",  
					},  
					// header  
					elem.Div(  
						attrs.Props{  
							"class": "flex items-center justify-between p-4 md:p-5 border-b rounded-t dark:border-gray-600",  
						},  
						elem.H3(  
							attrs.Props{  
								"class": "text-xl font-semibold text-gray-900 dark:text-white",  
							},  
							elem.Text(bluemonday.StrictPolicy().Sanitize(m.Name)),  
						),  
						elem.Button( // close button  
							attrs.Props{  
								"class":           "text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 ms-auto inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white",  
								"data-modal-hide": modalName(m),  
							},  
							elem.Raw(  
								`<svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">  
									<path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"/>  
								</svg>`,  
							),  
							elem.Span(  
								attrs.Props{  
									"class": "sr-only",  
								},  
								elem.Text("Close modal"),  
							),  
						),  
					),  
					// body  
					elem.Div(  
						attrs.Props{  
							"class": "p-4 md:p-5 space-y-4",  
						},  
						elem.Div(  
							attrs.Props{  
								"class": "flex justify-center items-center",  
							},  
							elem.Img(attrs.Props{  
								//	"class": "rounded-t-lg object-fit object-center h-96",  
								"class":   "lazy rounded-t-lg max-h-48 max-w-96 object-cover mt-3 entered loaded",  
								"src":     m.Icon,  
								"loading": "lazy",  
							}),  
						),  
						elem.P(  
							attrs.Props{  
								"class": "text-base leading-relaxed text-gray-500 dark:text-gray-400",  
							},  
							elem.Text(bluemonday.StrictPolicy().Sanitize(m.Description)),  
						),  
						elem.Hr(  
							attrs.Props{},  
						),  
						elem.P(  
							attrs.Props{  
								"class": "text-sm font-semibold text-gray-900 dark:text-white",  
							},  
							elem.Text("Links"),  
						),  
						elem.Ul(  
							attrs.Props{},  
							urls...,  
						),  
						elem.If(  
							len(m.Tags) > 0,  
							elem.Div(  
								attrs.Props{},  
								elem.P(  
									attrs.Props{  
										"class": "text-sm mb-5 font-semibold text-gray-900 dark:text-white",  
									},  
									elem.Text("Tags"),  
								),  
								elem.Div(  
									attrs.Props{  
										"class": "flex flex-row flex-wrap content-center",  
									},  
									tagsNodes...,  
								),  
							),  
							elem.Div(attrs.Props{}),  
						),  
					),  
					// Footer  
					elem.Div(  
						attrs.Props{  
							"class": "flex items-center p-4 md:p-5 border-t border-gray-200 rounded-b dark:border-gray-600",  
						},  
						elem.Button(  
							attrs.Props{  
								"data-modal-hide": modalName(m),  
								"class":           "py-2.5 px-5 ms-3 text-sm font-medium text-gray-900 focus:outline-none bg-white rounded-lg border border-gray-200 hover:bg-gray-100 hover:text-blue-700 focus:z-10 focus:ring-4 focus:ring-gray-100 dark:focus:ring-gray-700 dark:bg-gray-800 dark:text-gray-400 dark:border-gray-600 dark:hover:text-white dark:hover:bg-gray-700",  
							},  
							elem.Text("Close"),  
						),  
					),  
				),  
			),  
		),  
	)  
}  
  
func modelActionItems(m *gallery.GalleryModel, processTracker ProcessTracker, galleryService *services.GalleryService) elem.Node {  
	galleryID := fmt.Sprintf("%s@%s", m.Gallery.Name, m.Name)  
	currentlyProcessing := processTracker.Exists(galleryID)  
	jobID := ""  
	isDeletionOp := false  
	if currentlyProcessing {  
		status := galleryService.GetStatus(galleryID)  
		if status != nil && status.Deletion {  
			isDeletionOp = true  
		}  
		jobID = processTracker.Get(galleryID)  
		// TODO:  
		// case not handled, if status == nil : "Waiting"  
	}  
  
	nodes := []elem.Node{  
		cardSpan("Repository: "+m.Gallery.Name, "fa-brands fa-git-alt"),  
	}  
  
	if m.License != "" {  
		nodes = append(nodes,  
			cardSpan("License: "+m.License, "fas fa-book"),  
		)  
	}  
	/*  
		tagsNodes := []elem.Node{}  
  
			for _, tag := range m.Tags {  
				tagsNodes = append(tagsNodes,  
					searchableElement(tag, "fas fa-tag"),  
				)  
			}  
  
  
				nodes = append(nodes,  
					elem.Div(  
						attrs.Props{  
							"class": "flex flex-row flex-wrap content-center",  
						},  
						tagsNodes...,  
					),  
				)  
  
				for i, url := range m.URLs {  
					nodes = append(nodes,  
						buttonLink("Link #"+fmt.Sprintf("%d", i+1), url),  
					)  
				}  
	*/  
  
	progressMessage := "Installation"  
	if isDeletionOp {  
		progressMessage = "Deletion"  
	}  
  
	return elem.Div(  
		attrs.Props{  
			"class": "px-6 pt-4 pb-2",  
		},  
		elem.P(  
			attrs.Props{  
				"class": "mb-4 text-base",  
			},  
			nodes...,  
		),  
		elem.Div(  
			attrs.Props{  
				"id":    "action-div-" + dropBadChars(galleryID),  
				"class": "flow-root", // To order buttons left and right  
			},  
			infoButton(m),  
			elem.Div(  
				attrs.Props{  
					"class": "float-right",  
				},  
				elem.If(  
					currentlyProcessing,  
					elem.Node( // If currently installing, show progress bar  
						elem.Raw(StartProgressBar(jobID, "0", progressMessage)),  
					), // Otherwise, show install button (if not installed) or display "Installed"  
					elem.If(m.Installed,  
						elem.Node(elem.Div(  
							attrs.Props{},  
							reInstallButton(m.ID()),  
							deleteButton(m.ID()),  
						)),  
						installButton(m.ID()),  
					),  
				),  
			),  
		),  
	)  
}  
  
func ListModels(models []*gallery.GalleryModel, processTracker ProcessTracker, galleryService *services.GalleryService) string {  
	modelsElements := []elem.Node{}  
  
	for _, m := range models {  
		elems := []elem.Node{}  
  
		if m.Icon == "" {  
			m.Icon = noImage  
		}  
  
		divProperties := attrs.Props{  
			"class": "flex justify-center items-center",  
		}  
  
		elems = append(elems,  
			elem.Div(divProperties,  
				elem.A(attrs.Props{  
					"href": "#!",  
					//		"class":     "justify-center items-center",  
				},  
					elem.Img(attrs.Props{  
						//	"class": "rounded-t-lg object-fit object-center h-96",  
						"class":   "rounded-t-lg max-h-48 max-w-96 object-cover mt-3",  
						"src":     m.Icon,  
						"loading": "lazy",  
					}),  
				),  
			),  
		)  
  
		// Special/corner case: if a model sets Trust Remote Code as required, show a warning  
		// TODO: handle this more generically later  
		_, trustRemoteCodeExists := m.Overrides["trust_remote_code"]  
		if trustRemoteCodeExists {  
			elems = append(elems, elem.Div(  
				attrs.Props{  
					"class": "flex justify-center items-center bg-red-500 text-white p-2 rounded-lg mt-2",  
				},  
				elem.I(attrs.Props{  
					"class": "fa-solid fa-circle-exclamation pr-2",  
				}),  
				elem.Text("Attention: Trust Remote Code is required for this model"),  
			))  
		}  
  
		elems = append(elems,  
			modelDescription(m),  
			modelActionItems(m, processTracker, galleryService),  
		)  
		modelsElements = append(modelsElements,  
			elem.Div(  
				attrs.Props{  
					"class": " me-4 mb-2 block rounded-lg shadow-secondary-1 dark:bg-surface-dark",  
				},  
				elem.Div(  
					attrs.Props{  
						//	"class": "p-6",  
					},  
					elems...,  
				),  
			),  
		)  
	}  
  
	wrapper := elem.Div(attrs.Props{  
		"class": "dark grid grid-cols-1 grid-rows-1 md:grid-cols-3 block rounded-lg shadow-secondary-1 dark:bg-surface-dark",  
	}, modelsElements...)  
  
	return wrapper.Render()  
}  
```  
  
# core/http/elements/p2p.go  
## Package: LocalAI/core/p2p  
  
### Imports:  
  
- fmt  
- github.com/chasefleming/elem-go  
- github.com/chasefleming/elem-go/attrs  
- github.com/microcosm-cc/bluemonday  
- github.com/mudler/LocalAI/core/p2p  
  
### External Data, Input Sources:  
  
- p2p.NodeData: This data structure represents a node in the peer-to-peer network. It contains information about the node, such as its ID, status, and other relevant details.  
  
### Code Summary:  
  
#### Function: renderElements  
  
- This function takes a slice of elem.Node objects as input and returns a string containing the rendered HTML representation of these nodes.  
- It iterates through the input slice and appends the rendered HTML of each node to a string variable called "render".  
- Finally, it returns the accumulated string containing the rendered HTML of all nodes.  
  
#### Function: P2PNodeStats  
  
- This function takes a slice of p2p.NodeData objects as input and returns a string containing the HTML representation of the P2P node statistics.  
- It first calculates the number of online nodes by iterating through the input slice and checking the IsOnline() method of each node.  
- Based on the number of online nodes, it sets a CSS class for the circle icon to either green (online) or red (offline).  
- It creates an elem.I object representing the circle icon with the appropriate CSS class and appends it to a slice of elem.Node objects.  
- It also creates a span element containing the number of online nodes and the total number of nodes, and appends it to the slice of elem.Node objects.  
- Finally, it calls the renderElements function to render the HTML representation of the nodes and returns the resulting string.  
  
#### Function: P2PNodeBoxes  
  
- This function takes a slice of p2p.NodeData objects as input and returns a string containing the HTML representation of the P2P node boxes.  
- It iterates through the input slice and creates a div element for each node, containing information about the node's ID, status, and other relevant details.  
- It uses the elem.If function to conditionally render the circle icon based on the node's online status.  
- It also uses the elem.Span function to create a span element containing the node's status (Online or Offline) based on its online status.  
- Finally, it appends the rendered div element for each node to a slice of elem.Node objects and calls the renderElements function to render the HTML representation of the nodes, returning the resulting string.  
  
# core/http/elements/progressbar.go  
## Package: elements  
  
### Imports:  
  
- github.com/chasefleming/elem-go  
- github.com/chasefleming/elem-go/attrs  
- github.com/microcosm-cc/bluemonday  
  
### External Data, Input Sources:  
  
- galleryID (string)  
- text (string)  
- showDelete (bool)  
- err (string)  
- galleryName (string)  
- progress (string)  
- uid (string)  
  
### DoneProgress Function:  
  
This function takes three arguments: galleryID, text, and showDelete. It returns a string that represents a progress bar with a label and a delete button if showDelete is true. The progress bar is created using the elem.Div and elem.H3 elements, and the text is sanitized using the bluemonday.StrictPolicy().Sanitize function.  
  
### ErrorProgress Function:  
  
This function takes two arguments: err and galleryName. It returns a string that represents an error message with an install button. The error message is created using the elem.Div and elem.H3 elements, and the error message is sanitized using the bluemonday.StrictPolicy().Sanitize function.  
  
### ProgressBar Function:  
  
This function takes one argument: progress. It returns a string that represents a progress bar with a given width. The progress bar is created using the elem.Div and elem.Div elements, and the width is set using the style attribute.  
  
### StartProgressBar Function:  
  
This function takes three arguments: uid, progress, and text. It returns a string that represents a progress bar with a label and a progress bar that updates every 600ms. The progress bar is created using the elem.Div and elem.H3 elements, and the text is sanitized using the bluemonday.StrictPolicy().Sanitize function. The progress bar updates using the hx-get attribute, which fetches the progress from the server every 600ms.  
  
  
  
