---
layout: post
title: "C#에서 NLog 메세지를 WPF의 TextBox에 출력하는 방법"
tags: [c#, WPF, logger, NLog]
comments: true
---

WPF는 C#으로 윈도우즈 애플리케이션을 개발하는데 있어 꽤 훌륭한 UI프레임워크이다. 그런데 NLog에서 기본으로 제공되는 타겟은 Winform의 RichTextBox에 대해서만 제공하고 있으며 WPF에 대해서는 기본적으로 제공하는 도구가 없다. 

NuGet의 WpgRichTextBox라는 wrapper라이브러리가 있으나 정작 사용해보면 잘 동작하지 않는다. 물론 윈도우의 시스템이벤트를 이용해서 로그를 전파하고 이를 수신해서 표시 할 수도 있으나, 시스템이벤트는 응용프로그램을 관리자권한으로 실행해야 하므로 보안상 고려하지 않는 것으로 하자. 

이 글에서는 NLog의 Target을 하나 만들어서 WPF 컴포넌트에 로그정보를 출력하는 방법을 소개하겟다.

이 방법은 NLog에서 기본적으로 제공하지 않는 Target에 대해 출력하는 모듈을 작성할 때 활용 할 수 있을 것으로 생각한다.

# 1. 사전준비

C#/WPF 프로젝트를 만들어서 버튼을 클릭하면 주기적으로 로그를 콘솔에 출력하는 프로그램을 작성한다.

## 1.1 WPF 프로젝트 생성

C# WPF 프로젝트를 생성하고 다음과 같이 TextBox와 Button을 하나씩 만들어본다.
각 컴포넌트의 이름은 TbLog와 BtnStart로 지정한다.

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABBUAAAJTCAIAAADYMrkbAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAB4GSURBVHhe7d3Pi7VpXh7wzs6t25B/YpjRzLQ6oyHqLpCdEE02IQsXImTjUrPIRkRnkSwMCU62EjWRgGHGLDTMtItJJEgwA50f9EAPYjCNCYKbpE7V9d51zn3V6fl2Pe3YVefz4aK6z7nueqpe6MVzddXL89ZfvfTXrkhdUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV3sh6elLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqYj88LXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUhf74WmpS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK62A9PS11Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1CV1SV1Sl9QldUldUpfUJXVJXVKX1MV+eFrqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQuqUvqkrqkLqlL6pK6pC6pS+qSuqQubwEAAEx94Qs/IiIiIiIi8m3z9ts/aD+IiIiIiMgo9oOIiIiIiExjP4iIiIiIyDT2g4iIiIiITHPaD5/97Bcu8/kr2Y6tbMdWtmMr27GV7djKdmxlO7ayHVvZjq1sx1a2YyvbsZXt2Mp2bGU7trIdW9mOrWzHVrZjK9uxle3YynZsZTu2sh1b2Y6tbMdWtmMr27GV7djKdmxlO7ayHVvZjq1sx1a2YyvbsZXt2Mp2bGU7trIdW9mOrWzHVrZjK9uxle3YynZsZTu2sh1b2Y6tbMdWtmMr27GV7djKdmxlO7ayHVvZjq1sx1a2YyvbsZXt2Mp2bGU7trIdW9mOrWzHVrZjK9uxle3YynZsZTu2sh1b2Y6tbMdWtmMr27GV7djKdmxlO7ayHVvZjq1sx1a2YyvbsZXt2Mp2bGU7trIdW9mOrWzHVrZjK9uxle3YynZsZTu2sh1b2Y6tbMdWtmMr27GV7djKdmxlO7ayHVvZjq1sx1a2YyvbsZXt2Mp2bGU7trIdW9mOrWzHVrZjK9uxle3YynZsZTu2sh1b2Y6tbMdWtmMr27GV7djKdmxlO7ayHVvZjp3ymc+87ecPIiIiIiIyit9fEhERERH5yPmBH/ibx/P5z//ww9U+//m9ekbuLvLmaj+8Vc/Lw9W22A8iIiIiIh85n/70Xz+ez33u8w9X+57veXurnpG7izxc7e6yW/W8PFxti/0gIiIiIvKRs91qPy/2g4iIiIjITWS71X5e7AcRERERkZvIdqv9vNgPIiIiIiI3ke1W+3mxH0REREREbiLbrfbzYj+87vzD3/jDP37/6//ix/b3RUREROTmst1qPy/2w5bTDfcHH3zr67/8d6u6yxe/+v4HH7z/H3/5x7b3L/ML77z/wX/7rZ+s98/yU7/xX/94P3P/pd/98k8+vvNwqdM3czr/bb/uE7EfRERERCTZbrWfF/thy8N++GC/j3/I6W7+49kPX/ip3/zDP95Wyv04ufzENRvsBxERERE5mO1W+3mxH7bc33C/98364cBZ9Zz7+E7d2d+tjve/+d7FqDh+928/iIiIiEiy3Wo/L/bDlocb7t/9+vsfvPtbP31RPfzE4Hf+0/sfz374kV/46t3XePwpx+knDO/+3u9c3O6ffiKxfxsfLfaDiIiIiCTbrfbzYj9syQ33P7+7c7/cCbndP/2U4PH9+7/GEI83+me/v5TfO/q5u+3xcOrsxwsXv+Z0+rp3V7hfEW9GRV/n/ut+2DXvcto5D+9/8P7Xf/PfXe6H059iefOFzi+el2c/ftl2joiIiIi80Gy32s+L/bDlzf+w3/5+wnp5vh9+4Z1316356V7/zfntvv/uTv381v/x1vyLX33/zaecrn///va5T93if9g1T5/+uGQe1sKb/XD6o52NorOXF3/Y+/cfN0mGzcMFRUREROTlZrvVfl7shy1v9sPl/3d/vH0/3w8XOftdo30/rMHwcOzi1jw394+XfTxw9w2sHx3UfnjymmcXTC6/xMVn3WV9z2cj4X7J/MEfvPnSa9g8fpaIiIiIvMhst9rPi/2w5ckb7sttcL4fTrfXp18FevD0frjYG2eXemjvJ8r5VHjz749DIifP98PT17y4+H0e/zj1WdU+jKW7b/7dL//iOnz/0i8viYiIiLyCbLfaz4v9sOVsP6x/P98MZ/9+/9tB6//Nnw5/1P3w5v/u333uNy+mwt0t++X/+D+/zvVrXkyO+1wuhIvPumjXzxnu/lCnS20vHz9FRERERF5qtlvt5+Wl7oe//ZO/vL37MeXslvoup7XwjT94d3vn4S58u1k/Gwbz/XD6ct/8+r/+8sXvCN3fu7/zO2d/kXq6Hy6/+TdV3jn7rs7a9Ud4+Pdf+er7D2dOl3r3t371zUsRERERefHZbrWflxexH7axcNoPf/+f/K/ztz6+bLfgp/vvi2e6XeyHy7+a/NF/f+kup99Weu+b51Ph4Vb+vfcu/tbybD/cV5d/9fnuu7r4Jh8/8f7l2dfdvpP7r/LN9947/0IiIiIi8oKz3Wo/Ly9iP2xj4Tu5H0630e+dvTzbD+d/+eHulv0f333iM/bD/eF1i5/c/2bU+Q83Lq7z4dc8tQ/f1Jvv6vzi91eO7Ys+fCeP39v9n24/IyIiIiIvNtut9vPygvfDw7s++uijjz766KOPPvro4/Dj3R32wY93N/p3Hz+uq93th7uPH9fVHj6uP+/KX+jPH0REREREXmfubq+Px+8viYiIiIjcRLZb7efFfhARERERuYlst9rPi/0gIiIiInIT+dznvnA83/d9P/Rwtbff/qGtel4ervb93/83tvefl4er2Q8iIiIiIjKN/SAiIiIiItM8sR/+zs/+7vlbIiIiIiIiD3ni+dPf+73f/0u/9MVf+7VfFxEREREROc/P/uw/utsLK5/+9GffupsR77zze/8PAADg0t2E2H/+YD8AAABPsh8AAIAp+wEAAJi6th/eSQ8AAPCG/QAAAEzZDwAAwJT9AAAATNkPAADAlP0AAABM2Q8AAMCU/QAAAExd2w+eHwcAAOwO7Ydf/Ve/KSIiIiIiryO5y/9QR/fD//if7/3pn/4fERERERF5ufnWt/7oO7Qf/uzP/iwvAACAl+nurt5+AAAARuwHAABg6hO2H/7vv/l7b333z/yHvAIAAD5R/hL2w2kjPOW0Gz50PxgXAADwl+v4fhg9P+7pnz/0ILAfAADgE8x+AAAApuwHAABg6pO5H07/iLP++n44O3/xGSd//p9//lNpvutH/9m//flPfdePfum/pwMAAD6KT+B+uLPevH/95tXT++F+HzxugsuXF5+fq9sPAADwTJ/A/XB5f3/aAznz5H544s311ulz+2r2AwAAPNMncD/0Gsgd/xPlk4NgvdefYD8AAMABn/j9cHbH/5z9sHX2AwAAHHB8Pxx4/vTT++Hy/v7szFP74ak37956uEavhb4+AAAw9gncD2d/w/k0AB5fPTEV7lyeeXj55tRld//K358GAIBn+wTuh+/+mX//cKN/cl4/jItzb9rzZpsHGQ1pvnH3yn4AAIBn+kvdD995pzVhPwAAwDPd1n4wHwAA4IjXvR/u9sIPPv4C1P1vOfVfoAAAAIZe/c8fLv7ShPEAAABHvPr9AAAAfGyO74cDz48DAABeFPsBAACYsh8AAIAp+wEAAJiyHwAAgCn7AQAAmLIfAACAKfsBAACYOr4fPD8OAABuhf0AAABM2Q8AAMCU/QAAAEzZDwAAwJT9AAAATNkPAADAlP0AAABMHd8Pnh8HAAC3wn4AAACm7AcAAGDKfgAAAKbsBwAAYMp+AAAApuwHAABgyn4AAACmju8Hz48DAIBbYT8AAABT9gMAADBlPwAAAFP2AwAAMGU/AAAAU/YDAAAwZT8AAABTx/eD58cBAMCtsB8AAIAp+wEAAJiyHwAAgCn7AQAAmLIfAACAKfsBAACYsh8AAICp4/vB8+MAAOBW2A8AAMCU/QAAAEzZDwAAwJT9AAAATNkPAADAlP0AAABM2Q8AAMDU8f3g+XEAAHAr7AcAAGDKfgAAAKbsBwAAYMp+AAAApuwHAABgyn4AAACm7AcAAGDq+H7w/DgAALgV9gMAADBlPwAAAFP2AwAAMGU/AAAAU/YDAAAwZT8AAABT9gMAADB1fD94fhwAANwK+wEAAJiyHwAAgCn7AQAAmLIfAACAKfsBAACYsh8AAIAp+wEAAJg6vh88Pw4AAG6F/QAAAEzZDwAAwJT9AAAATNkPAADAlP0AAABM2Q8AAMCU/QAAAEwd3w+eHwcAALfCfgAAAKbsBwAAYMp+AAAApuwHAABgyn4AAACm7AcAAGDKfgAAAKaO7wfPjwMAgFthPwAAAFP2AwAAMGU/AAAAU/YDAAAwZT8AAABT9gMAADBlPwAAAFPH94PnxwEAwK2wHwAAgCn7AQAAmLIfAACAKfsBAACYsh8AAIAp+wEAAJiyHwAAgKnj+8Hz4wAA4FbYDwAAwJT9AAAATNkPAADAlP0AAABM2Q8AAMCU/QAAAEzZDwAAwNTx/eD5cQAAcCvsBwAAYMp+AAAApuwHAABgyn4AAACm7AcAAGDKfgAAAKbsBwAAYOr4fvD8OAAAuBX2AwAAMGU/AAAAU/YDAAAwZT8AAABT9gMAADBlPwAAAFP2AwAAMHV8P3h+HAAA3Ar7AQAAmLIfAACAKfsBAACYsh8AAIAp+wEAAJiyHwAAgCn7AQAAmDq+Hzw/DgAAboX9AAAATNkPAADAlP0AAABM2Q8AAMCU/QAAAEzZDwAAwJT9AAAATB3fD54fBwAAt8J+AAAApuwHAABgyn4AAACm7AcAAGDKfgAAAKbsBwAAYMp+AAAApo7vB8+PAwCAW2E/AAAAU/YDAAAwZT8AAABT9gMAADBlPwAAAFP2AwAAMGU/AAAAU8f3g+fHAQDArbAfAACAKfsBAACYsh8AAIAp+wEAAJiyHwAAgCn7AQAAmLIfAACAqeP7wfPjAADgVtgPAADAlP0AAABM2Q8AAMCU/QAAAEzZDwAAwJT9AAAATNkPAADA1PH94PlxAABwK+wHAABgyn4AAACm7AcAAGDKfgAAAKbsBwAAYMp+AAAApuwHAABg6vh+8Pw4AAC4FfYDAAAwZT8AAABT9gMAADBlPwAAAFP2AwAAMGU/AAAAU/YDAAAwdXw/eH4cAADcCvsBAACYsh8AAIAp+wEAAJiyHwAAgCn7AQAAmLIfAACAKfsBAACYOr4fPD8OAABuhf0AAABM2Q8AAMCU/QAAAEzZDwAAwJT9AAAATNkPAADAlP0AAABMHd8Pnh8HAAC3wn4AAACm7AcAAGDKfgAAAKbsBwAAYMp+AAAApuwHAABgyn4AAACmju8Hz48DAIBbYT8AAABT9gMAADBlPwAAAFP2AwAAMGU/AAAAU/YDAAAwZT8AAABTx/eD58cBAMCtsB8AAIAp+wEAAJiyHwAAgCn7AQAAmLIfAACAKfsBAACYsh8AAICp4/vB8+MAAOBW2A8AAMCU/QAAAEzZDwAAwJT9AAAATNkPAADAlP0AAABM2Q8AAMDU8f3g+XEAAHAr7AcAAGDKfgAAAKbsBwAAYMp+AAAApuwHAABgyn4AAACm7AcAAGDq+H7w/DgAALgV9gMAADBlPwAAAFP2AwAAMGU/AAAAU/YDAAAwZT8AAABT9gMAADB1fD94fhwAANwK+wEAAJiyHwAAgCn7AQAAmLIfAACAKfsBAACYsh8AAIAp+wEAAJg6vh88Pw4AAG6F/QAAAEzZDwAAwJT9AAAATNkPAADAlP0AAABM2Q8AAMCU/QAAAEwd3w+eHwcAALfCfgAAAKbsBwAAYMp+AAAApuwHAABgyn4AAACm7AcAAGDKfgAAAKaO7wfPjwMAgFthPwAAAFP2AwAAMGU/AAAAU/YDAAAwZT8AAABT9gMAADBlPwAAAFPH94PnxwEAwK2wHwAAgCn7AQAAmLIfAACAKfsBAACYsh8AAIAp+wEAAJiyHwAAgKnj+8Hz4wAA4FbYDwAAwJT9AAAATNkPAADAlP0AAABM2Q8AAMCU/QAAAEzZDwAAwNTx/eD5cQAAcCvsBwAAYMp+AAAApuwHAABgyn4AAACm7AcAAGDKfgAAAKbsBwAAYOr4fvD8OAAAuBX2AwAAMGU/AAAAU/YDAAAwZT8AAABT9gMAADBlPwAAAFP2AwAAMHV8P3h+HAAA3Ar7AQAAmLIfAACAKfsBAACYsh8AAIAp+wEAAJiyHwAAgCn7AQAAmDq+Hzw/DgAAboX9AAAATNkPAADAlP0AAABM2Q8AAMCU/QAAAEzZDwAAwJT9AAAATB3fD54fBwAAt8J+AAAApuwHAABgyn4AAACm7AcAAGDKfgAAAKbsBwAAYMp+AAAApo7vB8+PAwCAW2E/AAAAU/YDAAAwZT8AAABT9gMAADBlPwAAAFP2AwAAMGU/AAAAU8f3g+fHAQDArbAfAACAKfsBAACYsh8AAIAp+wEAAJiyHwAAgCn7AQAAmLIfAACAqeP7wfPjAADgVtgPAADAlP0AAABM2Q8AAMCU/QAAAEzZDwAAwJT9AAAATNkPAADA1PH94PlxAABwK+wHAABgyn4AAACm7AcAAGDKfgAAAKbsBwAAYMp+AAAApuwHAABg6vh+8Pw4AAC4FfYDAAAwZT8AAABT9gMAADBlPwAAAFP2AwAAMGU/AAAAU/YDAAAwdXw/eH4cAADcCvsBAACYsh8AAIAp+wEAAJiyHwAAgCn7AQAAmLIfAACAKfsBAACYOr4fPD8OAABuhf0AAABM2Q8AAMCU/QAAAEzZDwAAwJT9AAAATNkPAADAlP0AAABMHd8Pnh8HAAC3wn4AAACm7AcAAGDKfgAAAKbsBwAAYMp+AAAApuwHAABgyn4AAACmju8Hz48DAIBbYT8AAABT9gMAADBlPwAAAFP2AwAAMGU/AAAAU/YDAAAwZT8AAABTx/eD58cBAMCtsB8AAIAp+wEAAJiyHwAAgCn7AQAAmLIfAACAKfsBAACYsh8AAICp4/vB8+MAAOBW2A8AAMCU/QAAAEzZDwAAwJT9AAAATNkPAADAlP0AAABM2Q8AAMDU8f3g+XEAAHAr7AcAAGDKfgAAAKbsBwAAYMp+AAAApuwHAABgyn4AAACm7AcAAGDq+H7w/DgAALgV9gMAADBlPwAAAFP2AwAAMGU/AAAAU/YDAAAwZT8AAABT9gMAADB1fD94fhwAANyK79x++Na3/ujPAQCAl+xP/uR/f4f2g4iIiIiIvI7kLv9DHdoPAADATbEfAACAKfsBAACYsh8AAIAp+wEAAJi6th9Gz48DAABuiv0AAABM2Q8AAC/eT8PHIf89fSj7AQDgxbu78/vKV77yX+C5vva1r9kPAAC34u7O7xvf+Ma78Fx3//3YDwAAt8J+4CD7AQDghtgPHGQ/AADcEPuBg47vB8+PAwB4MewHDrIfAABuiP3AQfYDAMANsR84yH4AALgh9gMH2Q8AADfEfuAg+wEA4IbYDxxkPwAA3BD7gYPsBwCAG2I/cNDx/eD5cQAAL4b9wEH2AwDADXl1++G3f+5Tb5378V9J8dF96Sf+Si7y1luf+rnfzrts7AcAgBvyKvfD483+/Zp4zoTYP/FLP3FgiLxu9gMAwA155fvh/ocI6+XeXXf60cO1wfBh3Yeaf/kXxX4AALghr30/XLy0H/4i2A8AADfkde+Hsxenf3308GaWwOkfDx5nwf17vRIej5686c/efdwH+doP3ad+/Mf7y78S9gMAwA15lfvhzMWd+sW2uJP7/vMZcHn/f94up2Nnb96dW6/uPyev7r/Ypx67/vKvhP0AAHBDXvfPH7ZNsN/AXw6Gh/5yLpze2UfEth8unH2F+0+9OLd/+Vfi+H7w/DgAgBfjte+Hi9v9vduXwLUb/NO5x4NP7If7A2+c7YfLi9kP9gMAwEv3+vfD2Rt7N90Pl83lZ52as0+6K698sQ+7/ItmPwAA3BA/fzi2H07F2SXOzvXFrl/+RbMfAABuyGvfD6eb/ceX22C4vh++9BMXN/oXV7n4CqcX6xqnYx/y+0v19V4H+wEA4Ia8yv1w7vJ+/U37cGN/fT/s17m8ysNOePPum1f3r+9eXN8P25d/JewHAIAb8ur2A99p9gMAwA2xHzjIfgAAuCH2Awcd3w+eHwcA8GLYDxxkPwAA3BD7gYPsBwCAG2I/cJD9AABwQ+wHDrIfAABuiP3AQfYDAMANsR84yH4AALgh9gMH2Q8AADfEfuCg4/vB8+MAAF4M+4GD7AcAgBtiP3CQ/QAAcEPu7vy+9rWv5U4QPrrf//3ftx8AAG7F3Z0fHJf/nj6U/QAAAEzZDwAAwJT9AAAATNkPAADA1LX94PlxAADAzn4AAACm7AcAAGDKfgAAAKbsBwAAYMp+AAAApuwHAABgyn4AAACmntgPf+sf/NMvfelfvvPo967kmu3YyjXbsZVrtmMr12zHVq7Zjq1csx1buWY7tnLNdmzlmu3YyjXbsZVrtmMr12zHVq7Zjq1csx1buWY7tnLNdmzlmu3YyjXbsZVrtmMr12zHVq7Zjq1csx1buWY7tnLNdmzlmu3YyjXbsZVrtmMr12zHVq7Zjq1csx1buWY7tnLNdmzlmu3YyjXbsZVrtmMr12zHVq7Zjq1csx1buWY7tnLNdmzlmu3YyjXbsZVrtmMr12zHVq7Zjq1csx1buWY7tnLNdmzlmu3YyjXbsZVrtmMr12zHVq7Zjq1csx1buWY7tnLNdmzlmu3YyjXbsZVrtmMr12zHVq7Zjq1csx1buWY7tnLNdmzl5Bd/8Yuf/ewXVj7zmc+dfv4gIiIiIiLybZPfXxIREREREfm2sR9ERERERGSat9/+wf8PO0oIob28huIAAAAASUVORK5CYII="/>


## 1.2 NLog 설치 및 환경설정

개발하고 있는 프로젝트에서 NuGet 패키지를 검색해서 설치하면 된다.

이후 App.config 파일을 열어서 다음과 같이 콘솔에 로그를 출력할 수 있도록 타겟과 룰을 하나 지정해 둔다.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <configSections>
      <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog"/>
    </configSections>
  
    <startup> 
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.6.1" />
    </startup>

    <nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <targets>
      <target name="logconsole" xsi:type="Console" />
    </targets>
    <rules>
      <logger name="*" minlevel="Trace" writeTo="logconsole" />
    </rules>
  </nlog>
</configuration>
```

## 1.3 버튼 이벤트 지정

디자인 편집기에서 버튼을 더블클릭해서 클릭이벤트 코드를 생성후 아래와 같이 3초마다 한번씩 반복적으로 로그를 출력하는 코드를 추가한다.

이제 프로그램을 실행 후 버튼을 클릭하면, 주기적으로 콘솔에 로그가 출력될 것이다.

```csharp
using System;
using System.Threading.Tasks;
using System.Windows;

namespace WpfNLog
{
    /// <summary>
    /// MainWindow.xaml에 대한 상호 작용 논리
    /// </summary>
    public partial class MainWindow : Window
    {
        private static readonly NLog.Logger Logger = NLog.LogManager.GetCurrentClassLogger();

        public MainWindow()
        {
            InitializeComponent();
        }

        private void BtnStart_Click(object sender, RoutedEventArgs e)
        {
            Task.Run(async ()=> {
                for(int i = 0; ; i++)
                {
                    await Task.Delay(TimeSpan.FromSeconds(3)).ConfigureAwait(false);
                    Logger.Debug("Log {0}", i);
                }
            });
        }
    }
}
```

# 2. TextBox에 로그 출력

Custom Target를 만들고, 이를 이용해서 TextBox에 로그메세지를 출력하는 것을 구현 해 본다.

## 2.1 로그이벤트를 전파할 Custom Target 생성

프로젝트에 NLogCustomTarget.cs파일을 새로 만든다음 다음과 같이 내용을 기록한다.
* TargetWithLayout을 상속받는 클래스를 생성하며, Target attribute에 해당 타겟의 이름을 적어준다.
* Write함수는 rule로 필터링 된 다음 해당 타겟에 로그를 기록할 때마다 불리는 함수이다. 이 안에서 로그이벤트를 전파하는 코드를 작성하면된다.
* delegate 호출시 null인 경우를 고려해서 ```?.Invoke``` 와 같은 형식으로 호출하는 편이 안전하다.

```csharp
using NLog;
using NLog.Targets;

namespace WpfNLog
{
    [Target("WpfTarget")]
    public class NLogCustomTarget : TargetWithLayout
    {
        // 로그 이벤트를 전파할 델리게이트 
        public delegate void LogEventDelegate(string message);
        public LogEventDelegate LogEventListener;

        // 생성자
        public NLogCustomTarget() { }

        // 로그 이벤트 기록
        protected override void Write(LogEventInfo logEvent)
        {
            // 레이아웃 형식에 맞게 로그 포맷
            string logMessage = this.Layout.Render(logEvent);
            // 로그이벤트를 수신할 델리게이트가 하나이상 지정된 경우에만 함수 호출
            LogEventListener?.Invoke(logMessage);
        }
    }
}
```

## 2.2 Custom Target 초기화 및 등록

Wpf의  Window Loaded 이벤트에 앞서 생성한 Target를 등록하는 코드를 작성한다.

그리고 다시 프로그램을 실행해서 BtnStart 버튼을 누르면 로그가 콘솔이 표시됨과 동시에 TextBox에도 출력되는 것을 볼 수 있다.

```csharp
private void Window_Loaded(object sender, RoutedEventArgs e)
{
    var config = new NLog.Config.LoggingConfiguration();

    // Targets where to log to: custom target
    var wpfLogger = new NLogCustomTarget();
    wpfLogger.LogEventListener += WriteLogToTextBox;

    // Rules for mapping loggers to targets
    config.AddRule(NLog.LogLevel.Debug, NLog.LogLevel.Fatal, wpfLogger);

    // Apply config           
    NLog.LogManager.Configuration = config;
}

private void WriteLogToTextBox(string log) {
    TbLog.Dispatcher.BeginInvoke(new Action(()=> {
        TbLog.AppendText(string.Format("\n{0}", log));
    }));
}
```


# 3. 참고 링크

* https://github.com/NLog/NLog/wiki/How-to-write-a-custom-target
* https://github.com/NLog/NLog/wiki/Tutorial